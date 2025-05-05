# SL-IL Platform Backend Documentation

## Project Overview

This is a Node.js/Express.js backend application that serves as the backend for the SL-IL Platform. The application uses MongoDB as its database and implements JWT-based authentication.

## Tech Stack

- Node.js
- Express.js
- MongoDB (Mongoose)
- JWT for authentication
- Nodemailer for email functionality
- Zod for validation
- Bcrypt for password hashing

## Project Structure

```
src/
├── app/
│   ├── DB/           # Database configuration and models
│   ├── errors/       # Custom error handling
│   │   └── AppError.js  # Custom error class
│   ├── middleware/   # Express middleware
│   │   ├── auth.js      # Authentication middleware
│   │   └── validateRequest.js # Zod validation middleware
│   ├── modules/      # Feature modules
│   │   ├── user/
│   │   │   ├── user.constant.js # User role enums
│   │   │   ├── user.controller.js
│   │   │   ├── user.model.js
│   │   │   ├── user.route.js
│   │   │   ├── user.service.js
│   │   │   └── user.validation.js
│   │   ├── auth/
│   │   ├── programs/
│   │   ├── learningMaterial/
│   │   ├── practical/
│   │   ├── assignment/
│   │   └── moduleRegistration.js # Module registration
│   ├── routes/       # Route definitions
│   │   └── index.js  # Root router
│   ├── utils/        # Utility functions
│   └── config.js     # Application configuration
├── app.js           # Express application setup
└── server.js        # Server entry point
```

## Core Module Architecture

Each feature module in the application follows a consistent architecture:

1. **Model**: Defines the Mongoose schema and model
2. **Controller**: Handles HTTP requests and responses
3. **Service**: Contains business logic and data access
4. **Route**: Defines the API endpoints for the feature
5. **Validation**: Contains Zod validation schemas
6. **Constants**: Defines constants related to the feature

## Authentication System

The application uses JWT (JSON Web Token) for authentication with a two-token system:

1. **Access Token**: Short-lived token for API authorization
2. **Refresh Token**: Long-lived token for obtaining new access tokens

Authentication flow:

1. User logs in with credentials
2. Server validates credentials
3. Server issues access and refresh tokens
4. Client stores tokens
5. Client includes access token in requests
6. When access token expires, client uses refresh token to get a new one

## Role-Based Access Control

The system implements role-based access control with the following roles:

```javascript
// From user.constant.js
export const UserRole = {
  superAdmin: "superAdmin",
  admin: "admin",
  user: "user",
};
```

Each API endpoint is protected by middleware that validates user roles.

## API Routes with Implementation Details

### Authentication Routes (/auth)

#### POST /auth/login

- **Controller**: AuthController.loginUser
- **Service**: AuthService.loginUser
- **Middleware**: validateRequest(LoginValidation)
- **Implementation**:
  - Validates user credentials
  - Generates JWT tokens
  - Returns user information and tokens

#### POST /auth/change-password

- **Controller**: AuthController.changePassword
- **Service**: AuthService.changePassword
- **Middleware**:
  - auth(UserRole.admin, UserRole.user)
  - validateRequest(ChangePasswordValidation)
- **Implementation**:
  - Validates old password
  - Hashes new password
  - Updates user record

#### POST /auth/refresh-token

- **Controller**: AuthController.refreshToken
- **Service**: AuthService.refreshToken
- **Implementation**:
  - Validates refresh token
  - Issues new access token

#### POST /auth/forget-password

- **Controller**: AuthController.forgetPassword
- **Service**: AuthService.forgetPassword
- **Middleware**: validateRequest(ForgetPasswordValidation)
- **Implementation**:
  - Validates email existence
  - Generates reset token
  - Sends email with reset link

#### POST /auth/create-password

- **Controller**: AuthController.createPassword
- **Service**: AuthService.createPassword
- **Middleware**: validateRequest(CreatePasswordValidation)
- **Implementation**:
  - Validates token
  - Sets initial password for user

#### POST /auth/reset-password

- **Controller**: AuthController.resetPassword
- **Service**: AuthService.resetPassword
- **Middleware**: validateRequest(ResetPasswordValidation)
- **Implementation**:
  - Validates reset token
  - Updates password in database

### User Routes (/users)

#### POST /users/create-user

- **Controller**: UserController.createUser
- **Service**: UserService.createUser
- **Middleware**:
  - auth(UserRole.admin, UserRole.superAdmin)
  - validateRequest(CreateUserValidation)
- **Implementation**:
  - Creates user with default password
  - Sends email with activation instructions

#### GET /users

- **Controller**: UserController.getUserProfile
- **Service**: UserService.getUserProfile
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin, UserRole.user)
- **Implementation**:
  - Extracts user ID from JWT
  - Returns user profile data

#### GET /users/:email

- **Controller**: UserController.getUserByEmail
- **Service**: UserService.getUserByEmail
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin)
- **Implementation**:
  - Finds user by email
  - Returns user data if found

#### PATCH /users/change-status

- **Controller**: UserController.changeUserStatus
- **Service**: UserService.changeUserStatus
- **Middleware**:
  - auth(UserRole.admin, UserRole.superAdmin)
  - validateRequest(ChangeStatusValidation)
- **Implementation**:
  - Updates user status
  - Returns updated user data

#### DELETE /users/:email

- **Controller**: UserController.deleteUser
- **Service**: UserService.deleteUser
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin)
- **Implementation**:
  - Performs soft delete (isDeleted: true)
  - Returns success message

#### PATCH /users/task-update

- **Controller**: UserController.updateUserTask
- **Service**: UserService.updateUserTask
- **Middleware**:
  - auth(UserRole.user)
  - validateRequest(TaskUpdateValidation)
- **Implementation**:
  - Adds task to user's completedTasks array
  - Returns updated task list

### Program Routes (/program)

#### POST /program/create-program

- **Controller**: ProgramController.createProgram
- **Service**: ProgramService.createProgramIntoDB
- **Middleware**:
  - auth(UserRole.admin, UserRole.superAdmin)
  - validateRequest(CreateProgramValidation)
- **Implementation**:
  - Creates new program with modules
  - Returns created program data

#### GET /program

- **Controller**: ProgramController.getAllPrograms
- **Service**: ProgramService.getAllPrograms
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin, UserRole.user)
- **Implementation**:
  - Role-based data access:
    - Users: Filtered data with completed tasks
    - Admins: Complete program data
  - Uses aggregation pipelines for user data
  - Returns programs with populated relations

Key implementation detail from `getAllPrograms` service:

```javascript
// From program.service.js
const getAllPrograms = async (role, info, email) => {
  let result;
  if (role === UserRole.user) {
    if (info.info) {
      const user = await User.isUserExistByEmail(email);

      if (!user) {
        throw new AppError(httpStatus.NOT_FOUND, "This user is not found !");
      }

      const completedTasks = user.completedTask;
      result = await Program.aggregate(
        getProgramDetailsAggregation(completedTasks)
      );
    } else {
      result = await Program.aggregate(getFullProgramAggregation);
    }
  } else {
    result = await Program.find().populate({
      path: "learningMaterials.learningMaterial",
      match: { isDeleted: false },
      select: "name description courseImage courseContents",
      // Additional population logic...
    });
  }
  return result;
};
```

### Learning Material Routes (/material)

#### POST /material/create

- **Controller**: LearningMaterialController.createLearningMaterial
- **Service**: LearningMaterialService.createLearningMaterialIntoDB
- **Middleware**:
  - auth(UserRole.admin, UserRole.superAdmin)
  - validateRequest(CreateLearningMaterialValidation)
- **Implementation**:
  - Creates new learning material
  - Returns created material data

#### GET /material/:programId

- **Controller**: LearningMaterialController.getLearningMaterialsByProgram
- **Service**: LearningMaterialService.getAllLearningMaterials
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin, UserRole.user)
- **Implementation**:
  - Role-based data access:
    - Users: Filtered data using aggregation
    - Admins: Complete material data
  - Returns materials with nested content structure

Key implementation detail from `getAllLearningMaterials` service:

```javascript
// From learningMaterial.service.js
const getAllLearningMaterials = async (role) => {
  let result;
  if (role === UserRole.user) {
    result = await LearningMaterial.aggregate([
      {
        $match: { isDeleted: false },
      },
      // Complex aggregation pipeline that:
      // 1. Filters deleted content
      // 2. Restructures content for frontend consumption
      // 3. Orders content by sortOrder
      // ...
    ]);
  } else {
    result = await LearningMaterial.find();
  }
  return result;
};
```

#### GET /material/single/:id

- **Controller**: LearningMaterialController.getLearningMaterialById
- **Service**: LearningMaterialService.getAllLearningMaterialById
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin, UserRole.user)
- **Implementation**:
  - Role-based data access
  - Returns single material with nested content

### Practical Routes (/practical)

#### POST /practical/create

- **Controller**: PracticalController.createPractical
- **Service**: PracticalService.createPracticalIntoDB
- **Middleware**:
  - auth(UserRole.admin, UserRole.superAdmin)
  - validateRequest(CreatePracticalValidation)
- **Implementation**:
  - Creates new practical material
  - Returns created practical data

#### GET /practical/:programId

- **Controller**: PracticalController.getPracticalsByProgram
- **Service**: PracticalService.getAllPracticals
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin, UserRole.user)
- **Implementation**:
  - Role-based data access
  - Returns practicals with nested content

### Assignment Routes (/assignment)

#### POST /assignment/create

- **Controller**: AssignmentController.createAssignment
- **Service**: AssignmentService.createAssignmentIntoDB
- **Middleware**:
  - auth(UserRole.admin, UserRole.superAdmin)
  - validateRequest(CreateAssignmentValidation)
- **Implementation**:
  - Creates new assignment
  - Returns created assignment data

#### GET /assignment/:programId

- **Controller**: AssignmentController.getAssignmentsByProgram
- **Service**: AssignmentService.getAllAssignments
- **Middleware**: auth(UserRole.admin, UserRole.superAdmin, UserRole.user)
- **Implementation**:
  - Role-based data access
  - Returns assignments with nested content

## Data Models

### User Model

- **Fields**: name, email, password, role, status, completedTask, isDeleted
- **Methods**: isUserExist, isUserExistByEmail, isPasswordMatched
- **Virtuals**: id
- **Statics**: findByEmail

### Program Model

- **Fields**: name, description, learningMaterials, practicals, assignments, isDeleted
- **Virtuals**: id
- **Population**: learningMaterials, practicals, assignments

### LearningMaterial Model

- **Fields**: name, description, courseImage, courseContents, isDeleted
- **Virtuals**: id
- **Population**: courseContents.contentDetails

### Practical Model

- **Fields**: name, courseImage, courseContents, isDeleted
- **Virtuals**: id
- **Population**: courseContents.contentDetails

### Assignment Model

- **Fields**: name, description, courseImage, isDeleted
- **Virtuals**: id

## Middleware Implementation

### Authentication Middleware (auth.js)

- **Purpose**: Protects routes based on user roles
- **Implementation**:
  - Extracts JWT from Authorization header
  - Verifies token validity
  - Checks user role against allowed roles
  - Injects user object into request
  - Blocks unauthorized access

```javascript
// Simplified auth middleware
const auth = (...requiredRoles) => {
  return async (req, res, next) => {
    try {
      // Extract and verify token
      const token = req.headers.authorization;
      if (!token) {
        throw new AppError(httpStatus.UNAUTHORIZED, "Unauthorized access");
      }

      // Decode token and get user
      const decoded = jwtHelpers.verifyToken(token);
      const { email } = decoded;

      // Find user and check role
      const user = await User.isUserExistByEmail(email);
      if (!user) {
        throw new AppError(httpStatus.NOT_FOUND, "User not found");
      }

      const { role } = user;
      if (requiredRoles.length && !requiredRoles.includes(role)) {
        throw new AppError(httpStatus.FORBIDDEN, "Forbidden access");
      }

      // Inject user into request
      req.user = user;
      next();
    } catch (error) {
      next(error);
    }
  };
};
```

### Validation Middleware (validateRequest.js)

- **Purpose**: Validates request data against Zod schemas
- **Implementation**:
  - Receives Zod schema as parameter
  - Validates request body against schema
  - Passes validation errors to error handler

```javascript
// Simplified validate middleware
const validateRequest = (schema) => {
  return async (req, res, next) => {
    try {
      await schema.parseAsync(req.body);
      next();
    } catch (error) {
      next(error);
    }
  };
};
```

## Frontend Overview

The frontend is a React-based application built with Vite, using modern React practices and a comprehensive set of tools for state management and UI components.

### Tech Stack

- React 18
- Vite
- Redux Toolkit for state management
- React Router for routing
- React Bootstrap for UI components
- React Hook Form for form handling
- Zod for validation
- Axios for API calls
- JWT for authentication
- SweetAlert2 for notifications
- React Toastify for toast notifications

### Key Components Structure

#### Authentication Components

- `Login.jsx` - User login form
- `GeneratePassword.jsx` - Initial password creation
- `ForgotPassword.jsx` - Password recovery request
- `ResetPassword.jsx` - Password reset form
- `ProtectedRoute.jsx` - Route protection wrapper

#### Layout Components

- `Layout.jsx` - Main application layout
- `Sidebar.jsx` - Navigation sidebar
- `Header.jsx` - Application header
- `Footer.jsx` - Application footer

#### User Dashboard Components

- `UserDashboard.jsx` - User's main dashboard
- `UserProfile.jsx` - User profile management
- `CourseListing.jsx` - List of available courses
- `CourseDetails.jsx` - Course detail view
- `CourseContent.jsx` - Course content display
- `TaskManager.jsx` - User task tracking

#### Admin Components

- `AdminDashboard.jsx` - Admin main dashboard
- `UserManagement.jsx` - User management interface
- `ProgramManagement.jsx` - Program management interface
- `ContentCreation.jsx` - Content creation tools
- `AssignmentManagement.jsx` - Assignment management

### Frontend Route Structure with Backend Integration

#### Authentication Routes

##### `/` - Login Page

- **Component**: `Login.jsx`
- **Redux Actions**: `login`, `clearAuthError`
- **API Integration**: `POST /auth/login`
- **State Management**: Stores JWT tokens and user info in Redux and localStorage
- **Protected**: No

##### `/generatePassword` - Password Generation

- **Component**: `GeneratePassword.jsx`
- **Redux Actions**: `createPassword`
- **API Integration**: `POST /auth/create-password`
- **Query Params**: `token` - JWT token from email
- **Protected**: No

##### `/forgotPassword` - Password Recovery

- **Component**: `ForgotPassword.jsx`
- **Redux Actions**: `forgotPassword`
- **API Integration**: `POST /auth/forget-password`
- **Protected**: No

##### `/resetPassword` - Password Reset

- **Component**: `ResetPassword.jsx`
- **Redux Actions**: `resetPassword`
- **API Integration**: `POST /auth/reset-password`
- **Query Params**: `token` - Reset token from email
- **Protected**: No

#### User Routes

##### `/home` - User Dashboard

- **Component**: `UserDashboard.jsx`
- **Redux Actions**: `fetchPrograms`
- **API Integration**: `GET /program`
- **State Management**: Stores programs in Redux
- **Protected**: Yes (UserRole.user)

##### `/home/profile` - User Profile

- **Component**: `UserProfile.jsx`
- **Redux Actions**: `fetchUserProfile`, `updateUserProfile`
- **API Integration**: `GET /users`, `PATCH /users`
- **Protected**: Yes (UserRole.user)

##### `/home/courses/regular` - Course Listing

- **Component**: `CourseListing.jsx`
- **Redux Actions**: `fetchPrograms`
- **API Integration**: `GET /program`
- **Implementation**: Displays list of available programs
- **Protected**: Yes (UserRole.user)

##### `/home/courses/regular/:id` - Course Details

- **Component**: `CourseDetails.jsx`
- **Redux Actions**: `fetchLearningMaterials`
- **API Integration**: `GET /material/:programId`
- **URL Params**: `id` - Program ID
- **Implementation**: Displays learning materials for a program
- **Protected**: Yes (UserRole.user)

##### `/home/courses/course/:id` - Course Content

- **Component**: `CourseContent.jsx`
- **Redux Actions**: `fetchLearningMaterial`, `markTaskComplete`
- **API Integration**:
  - `GET /material/single/:id`
  - `PATCH /users/task-update`
- **URL Params**: `id` - Learning Material ID
- **Implementation**: Displays detailed learning material content
- **Protected**: Yes (UserRole.user)

#### Admin Routes

##### `/admin` - Admin Dashboard

- **Component**: `AdminDashboard.jsx`
- **Redux Actions**: `fetchAdminStats`
- **API Integration**: Multiple admin endpoints
- **Protected**: Yes (UserRole.admin, UserRole.superAdmin)

##### `/admin/create-user` - User Creation

- **Component**: `CreateUser.jsx`
- **Redux Actions**: `createUser`
- **API Integration**: `POST /users/create-user`
- **Protected**: Yes (UserRole.admin, UserRole.superAdmin)

##### `/admin/user-info` - User Information

- **Component**: `UserInfo.jsx`
- **Redux Actions**: `fetchUsers`, `deleteUser`, `changeUserStatus`
- **API Integration**:
  - `GET /users/:email`
  - `DELETE /users/:email`
  - `PATCH /users/change-status`
- **Protected**: Yes (UserRole.admin, UserRole.superAdmin)

##### `/admin/profile` - Admin Profile

- **Component**: `AdminProfile.jsx`
- **Redux Actions**: `fetchUserProfile`, `updateAdminProfile`
- **API Integration**: `GET /users`, `PATCH /users`
- **Protected**: Yes (UserRole.admin, UserRole.superAdmin)

### Redux State Management

#### Auth Slice

- **State**: `{user, token, isAuthenticated, loading, error}`
- **Actions**: `login`, `logout`, `refreshToken`, `forgotPassword`, `resetPassword`
- **Selectors**: `selectUser`, `selectIsAuthenticated`, `selectAuthError`

#### User Slice

- **State**: `{users, currentUser, loading, error}`
- **Actions**: `fetchUsers`, `fetchUserProfile`, `createUser`, `updateUser`, `deleteUser`
- **Selectors**: `selectUsers`, `selectCurrentUser`, `selectUserLoading`

#### Program Slice

- **State**: `{programs, currentProgram, materials, loading, error}`
- **Actions**: `fetchPrograms`, `fetchProgram`, `fetchLearningMaterials`
- **Selectors**: `selectPrograms`, `selectCurrentProgram`, `selectLearningMaterials`

#### Task Slice

- **State**: `{tasks, completedTasks, loading, error}`
- **Actions**: `fetchTasks`, `markTaskComplete`, `fetchCompletedTasks`
- **Selectors**: `selectTasks`, `selectCompletedTasks`, `selectTaskLoading`

### API Integration Layer

The frontend uses Axios for API communication:

```javascript
// API client setup
import axios from "axios";

const baseURL = import.meta.env.VITE_API_BASE_URL;

const apiClient = axios.create({
  baseURL,
  timeout: 10000,
  headers: {
    "Content-Type": "application/json",
  },
});

// Request interceptor for adding auth token
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");
    if (token) {
      config.headers.Authorization = `${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor for error handling
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    // Handle token expiration
    if (error.response && error.response.status === 401) {
      // Try to refresh token or logout
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

### Authentication and Route Protection

The application uses a `ProtectedRoute` component to guard routes:

```jsx
// ProtectedRoute implementation
const ProtectedRoute = ({ element, allowedRoles }) => {
  const { isAuthenticated, user } = useSelector(selectAuth);
  const navigate = useNavigate();
  const dispatch = useDispatch();

  useEffect(() => {
    if (!isAuthenticated) {
      navigate("/");
    } else if (allowedRoles && !allowedRoles.includes(user.role)) {
      dispatch(logout());
      navigate("/");
    }
  }, [isAuthenticated, user, allowedRoles, navigate, dispatch]);

  return isAuthenticated ? element : null;
};
```

## Data Flow Between Frontend and Backend

### User Authentication Flow

1. User enters credentials in `Login.jsx`
2. Frontend dispatches `login` action
3. API request sent to `POST /auth/login`
4. Backend validates credentials, returns tokens and user data
5. Frontend stores tokens and user data in Redux and localStorage
6. Frontend redirects to appropriate dashboard based on user role

### Course Navigation Flow

1. User accesses `/home/courses/regular`
2. `CourseListing.jsx` component mounts
3. Component dispatches `fetchPrograms` action
4. API request sent to `GET /program`
5. Backend processes request through:
   - `auth` middleware checks JWT token
   - `ProgramController.getAllPrograms` handles request
   - `ProgramService.getAllPrograms` retrieves data with role-based filtering
   - Response sent to frontend
6. Frontend receives program data and renders course list
7. User clicks on a course
8. Frontend navigates to `/home/courses/regular/:id`
9. `CourseDetails.jsx` component mounts
10. Component dispatches `fetchLearningMaterials` action
11. Backend serves learning materials for the selected program
12. Frontend renders learning materials

### User Task Completion Flow

1. User completes a task in course content
2. Frontend dispatches `markTaskComplete` action
3. API request sent to `PATCH /users/task-update`
4. Backend updates user's completedTask array
5. Success response sent to frontend
6. Frontend updates UI to reflect completed state

## Error Handling

### Backend Error Handling

The backend uses a centralized error handler with custom error classes:

```javascript
// AppError.js - Custom error class
class AppError extends Error {
  constructor(statusCode, message) {
    super(message);
    this.statusCode = statusCode;
    this.status = statusCode >= 400 && statusCode < 500 ? "error" : "fail";

    Error.captureStackTrace(this, this.constructor);
  }
}

// Global error handler middleware
const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;

  // Log error in development
  if (process.env.NODE_ENV === "development") {
    console.error(err);
  }

  // Handle specific error types
  if (err.name === "CastError") {
    error = new AppError(400, "Invalid ID format");
  }

  // Send standardized error response
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || "Internal server error",
    error: {
      code: error.code || "UNKNOWN_ERROR",
      details: error.details || null,
    },
  });
};
```

### Frontend Error Handling

The frontend handles errors through:

1. Redux action error handling
2. API interceptors for global error management
3. Component-level error boundaries
4. Toast notifications for user feedback

## Security Implementation

### JWT Authentication

```javascript
// JWT helpers
import jwt from "jsonwebtoken";
import config from "../config";

const createToken = (payload, secret, expiry) => {
  return jwt.sign(payload, secret, {
    expiresIn: expiry,
  });
};

const verifyToken = (token) => {
  return jwt.verify(token, config.jwt.secret);
};

// Generate auth tokens
const generateTokens = (user) => {
  const accessToken = createToken(
    { email: user.email, role: user.role },
    config.jwt.secret,
    config.jwt.expires_in
  );

  const refreshToken = createToken(
    { email: user.email },
    config.jwt.refresh_secret,
    config.jwt.refresh_expires_in
  );

  return {
    accessToken,
    refreshToken,
  };
};
```

### Password Security

```javascript
// Password hashing
import bcrypt from "bcrypt";
import config from "../config";

const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(Number(config.bcrypt_salt_rounds));
  return await bcrypt.hash(password, salt);
};

const comparePassword = async (givenPassword, savedPassword) => {
  return await bcrypt.compare(givenPassword, savedPassword);
};
```

## Development Best Practices

### API Response Format

All API responses follow a standard format:

```javascript
// Success response
res.status(200).json({
  success: true,
  message: "Operation successful",
  data: result,
});

// Error response
res.status(400).json({
  success: false,
  message: "Operation failed",
  error: {
    code: "VALIDATION_ERROR",
    details: errors,
  },
});
```

### Validation with Zod

```javascript
// Validation schema example
import { z } from "zod";

const loginValidationSchema = z.object({
  email: z.string().email("Invalid email format"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});

// Controller with validation
const login = async (req, res, next) => {
  try {
    // Validation happens in middleware
    const { email, password } = req.body;
    const result = await AuthService.loginUser(email, password);

    res.status(200).json({
      success: true,
      message: "User logged in successfully",
      data: result,
    });
  } catch (error) {
    next(error);
  }
};
```

### Modular Architecture

Both frontend and backend use modular architectures:

- Backend: Feature-based modules with MVC pattern
- Frontend: Component-based architecture with Redux

This ensures:

- Separation of concerns
- Reusability
- Maintainability
- Testability

## Deployment Considerations

### Backend Deployment

The backend is configured for deployment on Vercel (vercel.json present):

```json
{
  "version": 2,
  "builds": [
    {
      "src": "src/server.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "src/server.js",
      "methods": ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
    }
  ]
}
```

### Frontend Deployment

The frontend is configured for deployment with Nginx (nginx.conf present):

```
server {
    listen 80;
    server_name example.com;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend-service:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
  }
}
```

## API Data JSON Format

### Standard Response Format

All API endpoints follow a standardized response format:

```javascript
// Success response
{
  "success": true,
  "statusCode": 200,
  "message": "Operation successful",
  "data": { /* Response data */ }
}

// Error response
{
  "success": false,
  "statusCode": 400,
  "message": "Operation failed",
  "error": {
    "code": "ERROR_CODE",
    "details": { /* Error details */ }
  }
}
```

### Authentication Endpoints

#### Login Request/Response

**Request:**

```json
{
  "email": "user@example.com",
  "password": "Password123"
}
```

**Response:**

```json
{
  "success": true,
  "statusCode": 200,
  "message": "User logged in successfully",
  "data": {
    "user": {
      "id": "60d0fe4f5311236168a109ca",
      "name": "John Doe",
      "email": "user@example.com",
      "role": "user",
      "status": "active",
      "completedTask": ["task1", "task2"]
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

### User Management Endpoints

#### Create User Request/Response

**Request:**

```json
{
  "name": "New User",
  "email": "newuser@example.com",
  "role": "user"
}
```

**Response:**

```json
{
  "success": true,
  "statusCode": 201,
  "message": "User created successfully",
  "data": {
    "id": "60d0fe4f5311236168a109cb",
    "name": "New User",
    "email": "newuser@example.com",
    "role": "user",
    "status": "pending"
  }
}
```

### Program Endpoints

#### Get All Programs Response

**Response for Admin:**

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Programs retrieved successfully",
  "data": [
    {
      "id": "60d0fe4f5311236168a109cc",
      "name": "Web Development",
      "description": "Learn web development fundamentals",
      "learningMaterials": [
        {
          "id": "60d0fe4f5311236168a109cd",
          "name": "HTML Basics",
          "description": "Introduction to HTML",
          "courseImage": "html.jpg",
          "courseContents": [
            {
              "id": "60d0fe4f5311236168a109ce",
              "title": "HTML Elements",
              "content": "Learn about HTML elements",
              "sortOrder": 1
            }
          ]
        }
      ],
      "practicals": [
        {
          "id": "60d0fe4f5311236168a109cf",
          "name": "HTML Practice",
          "courseImage": "html-practice.jpg"
        }
      ],
      "assignments": [
        {
          "id": "60d0fe4f5311236168a109cg",
          "name": "HTML Assignment",
          "description": "Create a basic HTML page"
        }
      ]
    }
  ]
}
```

**Response for User:**

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Programs retrieved successfully",
  "data": [
    {
      "id": "60d0fe4f5311236168a109cc",
      "name": "Web Development",
      "description": "Learn web development fundamentals",
      "progress": 35,
      "materials": [
        {
          "id": "60d0fe4f5311236168a109cd",
          "name": "HTML Basics",
          "description": "Introduction to HTML",
          "completed": true,
          "image": "html.jpg"
        },
        {
          "id": "60d0fe4f5311236168a109ce",
          "name": "CSS Basics",
          "description": "Introduction to CSS",
          "completed": false,
          "image": "css.jpg"
        }
      ]
    }
  ]
}
```

### Learning Material Endpoints

#### Get Material By ID Response

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Learning material retrieved successfully",
  "data": {
    "id": "60d0fe4f5311236168a109cd",
    "name": "HTML Basics",
    "description": "Introduction to HTML",
    "courseImage": "html.jpg",
    "courseContents": [
      {
        "id": "60d0fe4f5311236168a109ce",
        "title": "HTML Elements",
        "contentDetails": {
          "type": "text",
          "content": "HTML elements are the building blocks of HTML pages."
        },
        "sortOrder": 1,
        "isCompleted": true
      },
      {
        "id": "60d0fe4f5311236168a109cf",
        "title": "HTML Attributes",
        "contentDetails": {
          "type": "video",
          "content": "https://example.com/video.mp4"
        },
        "sortOrder": 2,
        "isCompleted": false
      }
    ]
  }
}
```

### Task Update Endpoint

#### Update Task Request/Response

**Request:**

```json
{
  "taskId": "60d0fe4f5311236168a109ce"
}
```

**Response:**

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Task updated successfully",
  "data": {
    "completedTask": ["60d0fe4f5311236168a109ce", "60d0fe4f5311236168a109cd"]
  }
}
```

## Validation Rules

The API enforces various validation rules using Zod schemas:

### User Validation

```javascript
const createUserValidationSchema = z.object({
  name: z.string().min(3, "Name must be at least 3 characters"),
  email: z.string().email("Invalid email format"),
  role: z.enum([UserRole.admin, UserRole.user], "Invalid role"),
});
```

### Program Validation

```javascript
const createProgramValidationSchema = z.object({
  name: z.string().min(3, "Name must be at least 3 characters"),
  description: z.string().optional(),
  learningMaterials: z.array(z.string()).optional(),
  practicals: z.array(z.string()).optional(),
  assignments: z.array(z.string()).optional(),
});
```

### Learning Material Validation

```javascript
const createLearningMaterialValidationSchema = z.object({
  name: z.string().min(3, "Name must be at least 3 characters"),
  description: z.string().optional(),
  courseImage: z.string().optional(),
  courseContents: z.array(
    z.object({
      title: z.string(),
      contentDetails: z.object({
        type: z.enum(["text", "video", "image", "pdf"]),
        content: z.string(),
      }),
      sortOrder: z.number(),
    })
  ),
});
```

## Frontend Component Analysis

### Component Data Requirements

#### CourseUI Component

The CourseUI component displays a list of programs and requires the following data:

```javascript
// Expected data structure for CourseUI
{
  programs: [
    {
      id: string,
      name: string,
      description: string,
      progress: number, // Percentage of completion
      materials: [
        {
          id: string,
          name: string,
          description: string,
          completed: boolean,
          image: string,
        },
      ],
    },
  ];
}
```

Implementation details:

- Receives programs from Redux store
- Maps progress based on user's completedTask vs. total tasks
- Renders CourseTile components for each program
- Uses React Bootstrap Card components for UI

#### CourseTile Component

The CourseTile component displays a single program card and requires:

```javascript
// Expected props for CourseTile
{
  id: string,
  name: string,
  description: string,
  progress: number,
  image: string,
  onClick: function
}
```

Key implementation details:

- Renders a clickable card with program information
- Displays progress bar for completion status
- Uses React Router for navigation to course details
- Implements subtle hover effects for better UX

#### CourseDetails Component

The CourseDetails component displays detailed information about a program:

```javascript
// Expected data structure for CourseDetails
{
  program: {
    id: string,
    name: string,
    description: string
  },
  materials: [
    {
      id: string,
      name: string,
      description: string,
      courseImage: string,
      isCompleted: boolean
    }
  ],
  practicals: [
    {
      id: string,
      name: string,
      courseImage: string,
      isCompleted: boolean
    }
  ],
  assignments: [
    {
      id: string,
      name: string,
      description: string,
      isCompleted: boolean
    }
  ]
}
```

Key implementation details:

- Fetches program details using programId from URL params
- Uses tabs to organize learning materials, practicals, and assignments
- Renders cards for each type of content
- Tracks completion status with visual indicators

### Data Transformation Layer

The backend and frontend use different data structures, requiring transformation:

```javascript
// Backend service transformation example
const getAllPrograms = async (role, info, email) => {
  // ... existing code ...

  // Transformation for user role
  if (role === UserRole.user) {
    const user = await User.isUserExistByEmail(email);
    const completedTasks = user.completedTask;

    // Transform backend data to match frontend requirements
    result = await Program.aggregate([
      { $match: { isDeleted: false } },
      {
        $project: {
          id: "$_id",
          name: 1,
          description: 1,
          materials: {
            $map: {
              input: "$learningMaterials",
              as: "material",
              in: {
                id: "$$material.learningMaterial",
                name: "$$material.name",
                description: "$$material.description",
                completed: {
                  $in: ["$$material.learningMaterial", completedTasks],
                },
                image: "$$material.courseImage",
              },
            },
          },
          progress: {
            $multiply: [
              {
                $divide: [
                  {
                    $size: {
                      $setIntersection: ["$allTaskIds", completedTasks],
                    },
                  },
                  { $size: "$allTaskIds" },
                ],
              },
              100,
            ],
          },
        },
      },
    ]);
  }
  // ... existing code ...
};
```

### React Components with Redux Integration

```javascript
// Example of CourseUI component with Redux integration
const CourseUI = () => {
  const dispatch = useDispatch();
  const { programs, loading } = useSelector(selectPrograms);
  const { user } = useSelector(selectAuth);

  useEffect(() => {
    dispatch(fetchPrograms());
  }, [dispatch]);

  if (loading) return <LoadingSpinner />;

  return (
    <Container>
      <Row>
        {programs.map((program) => (
          <Col key={program.id} md={4} className="mb-4">
            <CourseTile
              id={program.id}
              name={program.name}
              description={program.description}
              progress={program.progress}
              image={program.materials[0]?.image || "default.jpg"}
              onClick={() => navigate(`/home/courses/regular/${program.id}`)}
            />
          </Col>
        ))}
      </Row>
    </Container>
  );
};

// Example of CourseDetails component
const CourseDetails = () => {
  const { id } = useParams();
  const dispatch = useDispatch();
  const { currentProgram, materials, practicals, assignments, loading } =
    useSelector(selectProgramDetails);

  useEffect(() => {
    if (id) {
      dispatch(fetchProgramDetails(id));
    }
  }, [id, dispatch]);

  // Component rendering logic
};
```

### Task Completion Flow

The frontend implements a task completion system that works as follows:

1. User views learning material in CourseContent component
2. Component displays a "Mark as Completed" button for each content item
3. When clicked, dispatches markTaskComplete action
4. Action sends PATCH request to /users/task-update
5. Backend updates user's completedTask array
6. Redux updates local state to reflect completion
7. UI updates to show completed status

```javascript
// Simplified task completion logic
const markAsComplete = (taskId) => {
  dispatch(markTaskComplete({ taskId }))
    .unwrap()
    .then(() => {
      toast.success("Task marked as complete!");
      // Update local UI immediately
      setCompletedTasks((prev) => [...prev, taskId]);
    })
    .catch((error) => {
      toast.error("Failed to update task status");
      console.error(error);
    });
};
```

## Conclusion

This documentation provides a comprehensive overview of the SL-IL Platform architecture, including:

- Backend routes, controllers, middleware, and validation
- Frontend routes, components, and state management
- Data flow between frontend and backend
- Security implementation details
- Development best practices
- Deployment configurations

Developers should use this as a reference when onboarding to understand the system architecture and implementation details.

```

```

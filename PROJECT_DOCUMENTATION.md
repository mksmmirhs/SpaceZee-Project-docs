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
│   ├── middleware/   # Express middleware
│   ├── modules/      # Feature modules
│   ├── routes/       # Route definitions
│   ├── utils/        # Utility functions
│   └── config.js     # Application configuration
├── app.js           # Express application setup
└── server.js        # Server entry point
```

## API Routes

### Authentication Routes (/auth)

#### POST /auth/login

- **Description**: User login endpoint
- **Request Body**:

```json
{
  "email": "user@example.com", // Required, valid email format
  "password": "userpassword" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User logged in successfully",
  "data": {
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token",
    "user": {
      "id": "user_id",
      "name": "User Name",
      "email": "user@example.com",
      "role": "user"
    }
  }
}
```

#### POST /auth/change-password

- **Description**: Change user password
- **Access**: Admin, User
- **Request Body**:

```json
{
  "oldPassword": "currentpassword", // Required, string
  "newPassword": "newpassword" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Password changed successfully",
  "data": null
}
```

#### POST /auth/refresh-token

- **Description**: Refresh JWT token
- **Request Cookies**:

```json
{
  "refreshToken": "refresh_token" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "data": {
    "accessToken": "new_jwt_token"
  }
}
```

#### POST /auth/forget-password

- **Description**: Initiate password reset
- **Request Body**:

```json
{
  "email": "user@example.com" // Required, valid email format
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Password reset email sent",
  "data": null
}
```

#### POST /auth/create-password

- **Description**: Create new password
- **Body**:
  - token: string
  - password: string
- **Response**: Success message

#### POST /auth/reset-password

- **Description**: Reset password with token
- **Body**:
  - token: string
  - newPassword: string
- **Response**: Success message

### User Routes (/users)

#### POST /users/create-user

- **Description**: Create new user
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "name": "User Name", // Required, string
  "email": "user@example.com", // Required, valid email format
  "role": "user" // Required, enum: ["admin", "user"]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "user_id",
    "name": "User Name",
    "email": "user@example.com",
    "role": "user",
    "status": "active"
  }
}
```

#### GET /users

- **Description**: Get current user profile
- **Access**: Admin, SuperAdmin, User
- **Response**: User profile data

#### GET /users/:email

- **Description**: Get user by email
- **Access**: Admin, SuperAdmin
- **Response**: User data

#### PATCH /users/change-status

- **Description**: Change user status
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "status": "in-progress" // Required, enum: ["in-progress", "blocked"]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User status updated successfully",
  "data": {
    "id": "user_id",
    "status": "in-progress"
  }
}
```

#### DELETE /users/:email

- **Description**: Delete user
- **Access**: Admin, SuperAdmin
- **Response**: Success message

#### PATCH /users/task-update

- **Description**: Update user's completed tasks
- **Access**: User
- **Request Body**:

```json
{
  "completedTask": "task_id" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Task updated successfully",
  "data": {
    "id": "user_id",
    "completedTasks": ["task_id"]
  }
}
```

### Program Routes (/program)

#### POST /program/create-program

- **Description**: Create new program
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "title": "Program Title", // Required, string
  "description": "Program Description", // Required, string
  "duration": 30, // Required, number
  "modules": [
    // Required, array
    {
      "title": "Module Title",
      "content": "Module Content"
    }
  ]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Program created successfully",
  "data": {
    "id": "program_id",
    "title": "Program Title",
    "description": "Program Description",
    "duration": 30,
    "modules": [...]
  }
}
```

#### POST /program

- **Description**: Get all programs
- **Access**: Admin, SuperAdmin, User
- **Response**: Array of programs

### Learning Material Routes (/material)

#### POST /material/create

- **Description**: Create learning material
- **Access**: Admin, SuperAdmin
- **Body**:
  - title: string
  - content: string
  - programId: string
  - type: string
- **Response**: Created material data

#### GET /material/:programId

- **Description**: Get materials by program
- **Access**: Admin, SuperAdmin, User
- **Response**: Array of materials

### Task Material Routes (/task)

#### POST /task/create

- **Description**: Create task
- **Access**: Admin, SuperAdmin
- **Body**:
  - title: string
  - description: string
  - programId: string
  - deadline: Date
- **Response**: Created task data

#### GET /task/:programId

- **Description**: Get tasks by program
- **Access**: Admin, SuperAdmin, User
- **Response**: Array of tasks

### JWT Routes (/jwt)

#### POST /jwt/verify

- **Description**: Verify JWT token
- **Body**:
  - token: string
- **Response**: Token validity status

#### POST /jwt/refresh

- **Description**: Refresh JWT token
- **Body**:
  - refreshToken: string
- **Response**: New access token

## Database Models

The application uses MongoDB with Mongoose ODM. Models are defined in the DB directory.

## Authentication

- JWT-based authentication
- Token-based session management
- Password hashing using bcrypt

## Error Handling

Custom error handling middleware for consistent error responses across the application.

## Security Features

- Password hashing
- JWT token authentication
- CORS configuration
- Input validation using Zod

## Development

To run the project locally:

1. Install dependencies: `npm install`
2. Set up environment variables
3. Run development server: `npm run dev`

## Production

The application is configured for deployment on Vercel (vercel.json present).

## Scripts

- `npm start`: Start the production server
- `npm run dev`: Start the development server with nodemon
- `npm run lint`: Run ESLint
- `npm run lint:fix`: Fix ESLint issues automatically

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

### Project Structure

```
frontend/
├── src/
│   ├── assets/        # Static assets
│   ├── components/    # Reusable React components
│   ├── hooks/         # Custom React hooks
│   ├── redux/         # Redux store and slices
│   ├── routes/        # Route definitions
│   ├── styles/        # CSS and styling files
│   ├── utils/         # Utility functions
│   ├── App.jsx        # Root component
│   └── main.jsx       # Application entry point
├── public/            # Public assets
└── index.html         # HTML template
```

### Key Features

1. **State Management**

   - Redux Toolkit for global state
   - Redux Persist for state persistence
   - Local storage integration

2. **Authentication**

   - JWT-based authentication
   - Protected routes
   - Token management

3. **UI Components**

   - Bootstrap-based components
   - Custom styled components
   - Responsive design
   - Toast notifications
   - Modal dialogs

4. **Form Handling**

   - React Hook Form integration
   - Form validation with Zod
   - Custom form components

5. **API Integration**
   - Axios for HTTP requests
   - API interceptors
   - Error handling

### Development

To run the frontend project locally:

1. Install dependencies: `npm install`
2. Start development server: `npm run dev`
3. Build for production: `npm run build`
4. Preview production build: `npm run preview`

### Deployment

The frontend is configured for deployment with Nginx (nginx.conf present).

## Full Stack Integration

The frontend and backend are designed to work together seamlessly:

- Backend provides RESTful APIs
- Frontend consumes these APIs using Axios
- JWT authentication is shared between both
- Consistent error handling across stack
- Shared validation schemas using Zod

## Development Workflow

1. Backend development:

   - API endpoints in Express
   - Database operations with Mongoose
   - Authentication middleware

2. Frontend development:
   - React components and hooks
   - Redux state management
   - API integration
   - UI/UX implementation

## Security Considerations

- JWT token management
- CORS configuration
- Input validation
- Secure password handling
- Protected routes
- API security

## Detailed API Endpoints

### Authentication Routes (/auth)

#### POST /auth/login

- **Description**: User login endpoint
- **Request Body**:

```json
{
  "email": "user@example.com", // Required, valid email format
  "password": "userpassword" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User logged in successfully",
  "data": {
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token",
    "user": {
      "id": "user_id",
      "name": "User Name",
      "email": "user@example.com",
      "role": "user"
    }
  }
}
```

#### POST /auth/change-password

- **Description**: Change user password
- **Access**: Admin, User
- **Request Body**:

```json
{
  "oldPassword": "currentpassword", // Required, string
  "newPassword": "newpassword" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Password changed successfully",
  "data": null
}
```

#### POST /auth/refresh-token

- **Description**: Refresh JWT token
- **Request Cookies**:

```json
{
  "refreshToken": "refresh_token" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "data": {
    "accessToken": "new_jwt_token"
  }
}
```

#### POST /auth/forget-password

- **Description**: Initiate password reset
- **Request Body**:

```json
{
  "email": "user@example.com" // Required, valid email format
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Password reset email sent",
  "data": null
}
```

#### POST /auth/create-password

- **Description**: Create new password
- **Body**:
  - token: string
  - password: string
- **Response**: Success message

#### POST /auth/reset-password

- **Description**: Reset password with token
- **Body**:
  - token: string
  - newPassword: string
- **Response**: Success message

### User Routes (/users)

#### POST /users/create-user

- **Description**: Create new user
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "name": "User Name", // Required, string
  "email": "user@example.com", // Required, valid email format
  "role": "user" // Required, enum: ["admin", "user"]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "user_id",
    "name": "User Name",
    "email": "user@example.com",
    "role": "user",
    "status": "active"
  }
}
```

#### GET /users

- **Description**: Get current user profile
- **Access**: Admin, SuperAdmin, User
- **Response**: User profile data

#### GET /users/:email

- **Description**: Get user by email
- **Access**: Admin, SuperAdmin
- **Response**: User data

#### PATCH /users/change-status

- **Description**: Change user status
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "status": "in-progress" // Required, enum: ["in-progress", "blocked"]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User status updated successfully",
  "data": {
    "id": "user_id",
    "status": "in-progress"
  }
}
```

#### DELETE /users/:email

- **Description**: Delete user
- **Access**: Admin, SuperAdmin
- **Response**: Success message

#### PATCH /users/task-update

- **Description**: Update user's completed tasks
- **Access**: User
- **Request Body**:

```json
{
  "completedTask": "task_id" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Task updated successfully",
  "data": {
    "id": "user_id",
    "completedTasks": ["task_id"]
  }
}
```

### Program Routes (/program)

#### POST /program/create-program

- **Description**: Create new program
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "title": "Program Title", // Required, string
  "description": "Program Description", // Required, string
  "duration": 30, // Required, number
  "modules": [
    // Required, array
    {
      "title": "Module Title",
      "content": "Module Content"
    }
  ]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Program created successfully",
  "data": {
    "id": "program_id",
    "title": "Program Title",
    "description": "Program Description",
    "duration": 30,
    "modules": [...]
  }
}
```

#### POST /program

- **Description**: Get all programs
- **Access**: Admin, SuperAdmin, User
- **Response**: Array of programs

### Learning Material Routes (/material)

#### POST /material/create

- **Description**: Create learning material
- **Access**: Admin, SuperAdmin
- **Body**:
  - title: string
  - content: string
  - programId: string
  - type: string
- **Response**: Created material data

#### GET /material/:programId

- **Description**: Get materials by program
- **Access**: Admin, SuperAdmin, User
- **Response**: Array of materials

### Task Material Routes (/task)

#### POST /task/create

- **Description**: Create task
- **Access**: Admin, SuperAdmin
- **Body**:
  - title: string
  - description: string
  - programId: string
  - deadline: Date
- **Response**: Created task data

#### GET /task/:programId

- **Description**: Get tasks by program
- **Access**: Admin, SuperAdmin, User
- **Response**: Array of tasks

### JWT Routes (/jwt)

#### POST /jwt/verify

- **Description**: Verify JWT token
- **Body**:
  - token: string
- **Response**: Token validity status

#### POST /jwt/refresh

- **Description**: Refresh JWT token
- **Body**:
  - refreshToken: string
- **Response**: New access token

## API Response Format

All API responses follow a standard format:

```json
{
  "success": boolean,
  "message": string,
  "data": object | array | null,
  "error": {
    "code": string,
    "message": string
  } | null
}
```

## Authentication

All protected routes require a valid JWT token in the Authorization header:

```
Authorization: Bearer <token>
```

## Error Handling

The API uses standard HTTP status codes:

- 200: Success
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Internal Server Error

## API Request/Response Formats

### Authentication Routes (/auth)

#### POST /auth/login

- **Description**: User login endpoint
- **Request Body**:

```json
{
  "email": "user@example.com", // Required, valid email format
  "password": "userpassword" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User logged in successfully",
  "data": {
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token",
    "user": {
      "id": "user_id",
      "name": "User Name",
      "email": "user@example.com",
      "role": "user"
    }
  }
}
```

#### POST /auth/change-password

- **Description**: Change user password
- **Access**: Admin, User
- **Request Body**:

```json
{
  "oldPassword": "currentpassword", // Required, string
  "newPassword": "newpassword" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Password changed successfully",
  "data": null
}
```

#### POST /auth/refresh-token

- **Description**: Refresh JWT token
- **Request Cookies**:

```json
{
  "refreshToken": "refresh_token" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "data": {
    "accessToken": "new_jwt_token"
  }
}
```

#### POST /auth/forget-password

- **Description**: Initiate password reset
- **Request Body**:

```json
{
  "email": "user@example.com" // Required, valid email format
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Password reset email sent",
  "data": null
}
```

### User Routes (/users)

#### POST /users/create-user

- **Description**: Create new user
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "name": "User Name", // Required, string
  "email": "user@example.com", // Required, valid email format
  "role": "user" // Required, enum: ["admin", "user"]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "user_id",
    "name": "User Name",
    "email": "user@example.com",
    "role": "user",
    "status": "active"
  }
}
```

#### PATCH /users/change-status

- **Description**: Change user status
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "status": "in-progress" // Required, enum: ["in-progress", "blocked"]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "User status updated successfully",
  "data": {
    "id": "user_id",
    "status": "in-progress"
  }
}
```

#### PATCH /users/task-update

- **Description**: Update user's completed tasks
- **Access**: User
- **Request Body**:

```json
{
  "completedTask": "task_id" // Required, string
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Task updated successfully",
  "data": {
    "id": "user_id",
    "completedTasks": ["task_id"]
  }
}
```

### Program Routes (/program)

#### POST /program/create-program

- **Description**: Create new program
- **Access**: Admin, SuperAdmin
- **Request Body**:

```json
{
  "title": "Program Title", // Required, string
  "description": "Program Description", // Required, string
  "duration": 30, // Required, number
  "modules": [
    // Required, array
    {
      "title": "Module Title",
      "content": "Module Content"
    }
  ]
}
```

- **Response**:

```json
{
  "success": true,
  "message": "Program created successfully",
  "data": {
    "id": "program_id",
    "title": "Program Title",
    "description": "Program Description",
    "duration": 30,
    "modules": [...]
  }
}
```

## Validation Rules

### Common Validation Rules

- Email: Must be a valid email format
- Password: String, max 20 characters
- IDs: Must be valid MongoDB ObjectId format
- Status: Must be one of the predefined enum values

### Role-Based Access

- SuperAdmin: Full access to all endpoints
- Admin: Access to user management and content creation
- User: Access to learning materials and tasks

## Error Response Format

```json
{
  "success": false,
  "message": "Error message",
  "error": {
    "code": "ERROR_CODE",
    "message": "Detailed error message"
  }
}
```

## Common Error Codes

- INVALID_INPUT: Validation error
- UNAUTHORIZED: Authentication required
- FORBIDDEN: Insufficient permissions
- NOT_FOUND: Resource not found
- INTERNAL_ERROR: Server error

## Frontend Routes

### Authentication Routes

- `/` - Login page
- `/generatePassword` - Generate new password
- `/forgotPassword` - Forgot password form
- `/resetPassword` - Reset password form

### User Routes (/home)

- `/home` - User dashboard with program cards
- `/home/profile` - User profile view
- `/home/certificate` - User certificates
- `/home/courses/regular` - List of regular courses
- `/home/courses/regular/:id` - Course details
- `/home/courses/course/:id` - Task manager for specific course

### Admin Routes (/admin)

- `/admin` - Admin dashboard
- `/admin/status-change` - Change user status
- `/admin/password-change` - Change user password
- `/admin/delete` - Delete user
- `/admin/profile` - Admin profile
- `/admin/create-user` - Create new user
- `/admin/user-info` - View user information

### Task Routes

#### EMI Task

- `/task/emi` - EMI calculator and management

#### Savings Account Task

- `/task/savings` - Savings account overview
- `/task/savings/savings-account` - Savings account creation
  - `/otp` - OTP verification
  - `/personal-details` - Personal information
  - `/address-details` - Address information
  - `/nominee-details` - Nominee information
  - `/professional-details` - Professional information
  - `/terms-and-conditions` - Terms acceptance

#### Demat Account Task

- `/task/demat-account` - Demat account creation
  - `/confirm-otp` - OTP verification
  - `/email` - Email verification
  - `/confirm-email-otp` - Email OTP verification
  - `/pan-details` - PAN details
  - `/segments` - Account segments
  - `/digilocker` - DigiLocker integration
  - `/profile` - Profile management
    - `/link-bank-account` - Bank account linking
    - `/documents` - Document upload
    - `/add-nominees` - Nominee details
    - `/last-step` - Final step
    - `/otp` - OTP verification
    - `/success` - Success page

#### Budgeting Task

- `/task/budgeting` - Budget planning and tracking

#### Spending Task

- `/task/spending` - Spending tracker and analysis

#### PAN Task

- `/task/pan` - PAN card management
  - `/epan-route` - E-PAN route selection
  - `/epan-adhar` - Aadhaar verification
  - `/epan-otp-acept` - OTP acceptance
  - `/epan-otp` - OTP verification
  - `/kyc` - KYC process
  - `/final` - Final submission

#### Bank Reconciliation Task

- `/task/bank_reconciliation` - Bank reconciliation statement

## Frontend Components Structure

### Layout Components

- `AuthLayout` - Authentication pages layout
- `UserLayout` - User dashboard layout
- `AdminLayout` - Admin dashboard layout
- `TaskLayout` - Task pages layout

### Authentication Components

- `LoginForm` - User login form
- `GeneratePasswordForm` - Password generation form
- `ForgetPasswordForm` - Password recovery form
- `ResetPasswordForm` - Password reset form

### User Components

- `UserProgramCard` - Program display cards
- `UserProfileView` - User profile view
- `UserCertificate` - Certificate display
- `CourseUI` - Course interface
- `CourseDetails` - Course details view
- `TaskManager` - Task management interface

### Admin Components

- `AdminProfile` - Admin profile management
- `AdminCreateUser` - User creation interface
- `AdminStatusChange` - User status management
- `AdminPasswordChange` - Password change interface
- `AdminDelete` - User deletion interface
- `AdminUserInfo` - User information view

### Task Components

- `UserEMIApp` - EMI calculator
- `DAApp` - Demat account application
- `PBDTApp` - Personal budget tracker
- `SpendingApp` - Spending tracker
- `PANApp` - PAN card management
- `Bank_Reconciliation` - Bank reconciliation interface

## Frontend Features

### Authentication

- JWT-based authentication
- Password recovery system
- Role-based access control
- Session management

### User Dashboard

- Program overview
- Course progress tracking
- Certificate management
- Profile management

### Admin Dashboard

- User management
- Program management
- Status control
- User creation and deletion

### Task Management

- Step-by-step task completion
- Document upload
- OTP verification
- Progress tracking
- Form validation
- Multi-step forms

### Integration Features

- DigiLocker integration
- Bank account linking
- PAN verification
- Aadhaar verification
- Email verification

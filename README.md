Authenticate Me Write Up

Backend
Used Sequelize to set up the backend database that will store user credentials

Used Express to create the server and handle all routes

Morgan middleware for logging info about requests and responses

Only allow CORS (cross origin resource sharing) in development using the cors middleware because the react frontend will be served from a different server than the express server. CORS isn’t needed in production because all of the React and Express resources will come from the same origin

Csurf middleware to prevent CSRF (cross site request forgery)

Csurf will add a _csrf cookie that is HTTP only to any server response

Also adds a method on all requests that will be set to another cookie (XSRF token). The XSRF token cookie value needs to be sent in the header of any request in all HTTP verbs besides GET

The header is used to make sure the request comes from your site and not an unauthorized site


Error handlers

Resource not found
Will catch any requests that don’t match any of the defined routes and create a server error with status code 404

Sequelize error handler
Will catch sequelize errors (that come from not conforming with the database specs) 
Applies for validation errors

Database constraints
Id
Not null, PK
Username
Not null, indexed, unique, max 30 char
Email
Not null, indexed, unique, max 256 char
Hashedpassword
Binary string, not null
Will use bcryptjs to hash the provided password and store it in the database
createdAt
Default value of now
updatedAt
Default value of now

Model scopes
Exclude sensitive information (hashedpassword) from the info that gets sent to the frontend

Three scopes

Default = just id and username
currentUser = everything except hashedPassword
loginUser = includes all fields, only used for authenticating password

Backend Login Flow
API login route will be hit with a request body holding a valid credential (either username or email) and password combination
The API login handler will look for a User with the input credential in either the username or email columns
Then the hashedPassword for that found User will be compared with the input password for a match
If there is a match, the API login route should send back a JWT in an HTTP only cookie and a response body. The JWT and the body will hold the user’s id, username, and email
Backend Signup Flow
API signup route will be hit with a request body holding a username, email and password
The API signup handler will create a User with the username, an email, and a hashedPassword (with bcrypt) created from the input password
If the creation is successful, the API signup route should send back a JWT in an HTTP-only cookie and a response body. The JWT and the body will hold the user’s id, username, and email
Backend Logout Flow
API logout route will be hit with a request
The API logout handler will remove the JWT cookie set by the login or signup API routes and return a JSON success message
User Authentication Middleware
setTokenCookie
Takes in the response and the session user and generates a JWT. the payload of the JWT will be the return instance of the .toSafeObject (username, id, email, but no password). 
restoreUser
Will verify and parse the JWT’s payload and search the database for a user with the id in the payload. If there is an error or the id cannot be found in the database, clear the cookie from the response
API Routes
Login: POST /api/session
Call the login static method from the User model
If there is a user returned, then call the setTokenCookie and return a JSON response with the user information
If no user returned, then create “login failed” error and invoke the next error handling middleware
Logout: DELETE /api/session
Remove token cookie and return a JSON success message
Signup: POST /api/users
Call the signup static method on the User model
If user is successfully created then call setTokenCookie and return a JSON response with the user info
If creation of user is unsuccessful, then sequelize validation error will be passed to the next error handling middleware
Get session user: GET /api/session
Call the restoreUser middleware
If valid user restored, will return the session user as JSON under the key of user
If not valid, will return empty JSON
Validating request bodies
Middleware to make sure each of the routes has the required content in the request bodies (ex signup should have valid email, username, password etc)



Front End
Uses Redux + React
Redux Store
BrowserRouter wrapped in Provider
Provider will provide the state based on the Redux store
Added a proxy of ‘http://localhost5001’ to force the frontend server to act like it’s being served from the backend server
This way, when I make a fetch request, it will be made to the backend, not the frontend server
csrfFetch will grab the XSRF token from the cookie and attach it to any request
Login Form page
Will have a session slice of state that includes the following
Id
Email
Username
createdAt
updatedAt
By default there is no user
Two POJO action creators
One to set the session user to the input parameter
The other to remove the session user
Signup Form page
Similar to the login form page, but instead of serving the backend with a request to authenticate an existing user, you’ll serve a request to create a user instead
Logout button
Will serve the backend with the DELETE /api/session route defined earlier
Navigation React Component
Will render navigation links and a logout button as an unordered list

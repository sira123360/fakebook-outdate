# Project Explanation

This document explains the structure and workflow of this React application.

## 1. Project Overview

This is a single-page application (SPA) built using **React**. It utilizes several common libraries and patterns in the React ecosystem:

*   **State Management**: **Redux** is used for managing global application state, such as user authentication status and user information.
*   **Routing**: **React Router** handles client-side navigation, allowing different components to be rendered based on the URL.
*   **UI Components**: **Ant Design** is used as a component library, providing pre-built and styled UI elements.

From its structure (features like a news feed, user profiles, and authentication), the application appears to be a social media or content-sharing platform.

## 2. Application Startup Process

The application initializes and loads as follows:

1.  **Entry Point (`src/index.js`)**:
    *   This is the first JavaScript file executed when the application loads.
    *   It renders the main `App` component into the DOM (`document.getElementById('root')`).
    *   It wraps the `App` component with:
        *   `<Provider store={store}>`: Makes the Redux store available to all components in the application.
        *   `<BrowserRouter>`: Enables client-side routing using React Router.
    *   It also imports global CSS files (`index.css` and Ant Design's CSS).

2.  **Redux Store Initialization (`src/redux/store/store.js`)**:
    *   A Redux store is created using `createStore` from Redux, combining all reducers (currently focused on `userReducer`).
    *   **State Persistence**: A key feature here is the use of `localStorage` for persisting the Redux state:
        *   `loadState()`: When the store initializes, this function attempts to retrieve the serialized state from `localStorage`. If found, this state is used as the initial state for the Redux store (pre-populating it). This is how user sessions can persist across browser closures.
        *   `saveState(state)`: The store is subscribed so that any time the state changes, this function is called to serialize the current state and save it back to `localStorage`.
    *   This persistence mechanism is crucial for keeping users logged in even after they close the browser tab or refresh the page.
## 3. State Management with Redux

Global application state is managed by Redux. This is particularly important for user authentication and user information.

1.  **Root Reducer (`src/redux/reducers/reducers.js`)**:
    *   This file uses `combineReducers` from Redux to merge all individual reducers into a single root reducer for the store.
    *   Currently, it primarily incorporates the `userReducer`.

2.  **User Reducer (`src/redux/reducers/userReducer.js`)**:
    *   This reducer is responsible for managing the `user` slice of the Redux state.
    *   **Initial State**:
        *   When the application loads, the `initialState` function of this reducer attempts to retrieve an `ACCESS_TOKEN` from `localStorage`.
        *   If a token is found, it's decoded using `jwt-decode`. The decoded payload (which typically contains user ID, role, name, etc.) becomes the initial user state.
        *   If no token is present in `localStorage`, the user state defaults to `{ role: "guest" }`.
    *   **Action Handling**:
        *   `LOGIN_USER`: When a `LOGIN_USER` action is dispatched (typically after a successful login API call), this reducer updates the `user` state with the action's payload (which includes `id`, `role`, `name`, and `profilePic`).
        *   `LOGOUT_USER`: When a `LOGOUT_USER` action is dispatched, this reducer resets the `user` state to `{ role: "guest" }`.

3.  **Actions (`src/redux/actions/actions.js`)**:
    *   This file defines action creators that facilitate state changes related to user authentication.
    *   **`login(user, token)`**:
        *   This action creator is called after a successful user login.
        *   It first calls `fetchLogin(token)`, which stores the provided `token` (JWT) into `localStorage` under the key "ACCESS_TOKEN" (or whatever `TOKEN` constant defines).
        *   It then returns an action object of type `LOGIN_USER`, with the `user` details (e.g., id, role, name, profilePic) as its payload.
    *   **`logout()`**:
        *   This action creator is called when the user logs out.
        *   It first calls `fetchLogout()`, which removes the "ACCESS_TOKEN" from `localStorage`.
        *   It then returns an action object of type `LOGOUT_USER`.
## 4. Routing Mechanism

Client-side routing is managed by React Router, allowing the application to navigate between different views without full page reloads.

1.  **Main Application Layout (`src/App.js`)**:
    *   The `App` component sets up the basic page structure using `Layout` from Ant Design (Header, Content).
    *   The `NavBar` component is always rendered in the Header.
    *   The main content area uses a `<Switch>` component from React Router to render different components based on the current URL.
    *   Crucially, it passes the `user.role` (obtained from the Redux store) as a prop to the `PrivateRoute` component.

2.  **Private Routes (`src/components/routes/PrivateRoute.js`)**:
    *   This component is the core of the application's authenticated routing logic.
    *   **Role-Based Access Control**:
        *   It receives the `role` of the current user as a prop from `App.js`.
        *   In its `componentDidMount` lifecycle method, it uses this `role` to look up allowed routes in the `rolesConfig` object (expected to be defined in `src/config/roles.js`). This configuration likely maps roles (e.g., "admin", "user", "guest") to an array of route objects, where each object specifies a `url` and the `component` to render.
        *   The `allowedRoutes` are stored in the component's state.
    *   **Route Rendering**:
        *   The `render` method iterates over the `allowedRoutes` array. For each allowed route, it creates a `<Route>` component from React Router, setting its `path` to `route.url` and `component` to the dynamically imported component (e.g., `allRoutes[route.component]`, where `allRoutes` is an aggregation of page components from `src/components/routes/index.js`).
    *   **Guest Redirection**:
        *   If the `user.role` prop is "guest", a `<Redirect to='/login' />` component is rendered, forcing any unauthenticated user trying to access a protected area to the login page.
        *   If the `role` prop is initially undefined (e.g., during the very first render cycle before Redux state is fully propagated), it attempts to redirect to `/login` programmatically using `this.props.history.push('/login')`.
## 5. Key Components

Several React components play crucial roles in the application's functionality:

1.  **Navigation Bar (`src/components/navbar/NavBar.js`)**:
    *   **User Display**: Connects to the Redux store (`mapStateToProps`) to access the current user's `name` and `profilePic`, displaying them in the navbar.
    *   **Navigation Links**:
        *   Provides a link to the home page via the application logo.
        *   Includes a dropdown menu (using Ant Design's `Dropdown` and `Menu`) with links to:
            *   "ดูรายชื่อเพื่อน" (View Friends List - `/friends`)
            *   "เปลี่ยนรหัสผ่าน" (Change Password - `/changepassword`)
            *   "ออกจากระบบ" (Logout)
    *   **Logout Process**:
        *   The "Logout" link triggers the `handleLogout` method.
        *   This method dispatches the `logout()` action (from `mapDispatchToProps`), which clears the access token from `localStorage` and resets the user state in Redux.
        *   It then redirects the user to the home page (`this.props.history.push('/')`) and forces a page reload (`window.location.reload(true)`), ensuring the guest state is fully applied.

2.  **Home Page (`src/pages/Home.js`)**:
    *   **User Context**: Connects to the Redux store to access the current user's information (`this.props.user`). In `componentDidMount`, it sets some of this user data (name, profilePic) into its local component state as `owner`.
    *   **Feed Fetching**:
        *   In `componentDidMount`, it uses `Axios` (pre-configured in `src/config/api.service.js`) to make a GET request to the `/feed` API endpoint.
        *   The response data (expected to be a list of posts) is stored in the component's local state variable `postList`.
    *   **Rendering**: The provided code snippet for `render` sets up a responsive column layout but does not yet include the logic to iterate over `postList` and display the actual posts. This part would need further implementation.

3.  **Login Page (`src/pages/authentication/Login.js` - Conceptual)**:
    *   While the code for `Login.js` was not explicitly reviewed, its role is critical for user authentication. It would typically:
        *   Provide input fields for username/email and password.
        *   On form submission, make an API request to a login endpoint with the user's credentials.
        *   If the API call is successful and returns user data along with an access token (JWT):
            *   Dispatch the `login(user, token)` action (from `src/redux/actions/actions.js`). This action stores the token in `localStorage` and updates the Redux store with the authenticated user's information.
        *   Upon successful login and Redux state update, the `PrivateRoute` component would then grant access to the appropriate authenticated routes.
        *   Handle login errors (e.g., display an error message for incorrect credentials).
## 6. Authentication Flow Summary

The application implements a token-based authentication flow, tightly integrated with Redux and `localStorage`.

1.  **User Login**:
    *   The user enters their credentials on the Login page (conceptually `src/pages/authentication/Login.js`).
    *   An API call is made to the backend authentication endpoint.
    *   If authentication is successful, the backend returns user details and a JSON Web Token (JWT) as an access token.
    *   The `login(user, token)` action (from `src/redux/actions/actions.js`) is dispatched.
        *   This action stores the JWT in `localStorage` (e.g., as "ACCESS_TOKEN").
        *   It updates the Redux store via `userReducer` with the authenticated user's information (ID, role, name, profile picture).
    *   With the Redux state updated, `PrivateRoute` grants access to routes permitted for the user's role.

2.  **Session Persistence**:
    *   When the application is loaded (or refreshed), the `initialState` of `userReducer` (`src/redux/reducers/userReducer.js`) checks `localStorage` for the "ACCESS_TOKEN".
    *   If the token exists, it is decoded using `jwt-decode`.
    *   The decoded information (user ID, role, name, etc.) is used to pre-populate the Redux `user` state.
    *   This makes the user appear "still logged in" without needing to re-enter credentials, as their session is restored from the persisted token.

3.  **User Logout**:
    *   The user clicks the "Logout" button (in `src/components/navbar/NavBar.js`).
    *   The `logout()` action (from `src/redux/actions/actions.js`) is dispatched.
        *   This action removes the "ACCESS_TOKEN" from `localStorage`.
        *   It updates the Redux store via `userReducer`, resetting the user state to `{ role: "guest" }`.
    *   The user is typically redirected to the login page (or home page, which then redirects to login due to the "guest" role and `PrivateRoute`'s logic). Access to protected routes is revoked.

# eWarrants: Enterprise Warranty Management System

![eWarrants Banner](https://res.cloudinary.com/djni7gwm4/image/upload/v1756023613/EWarrants_white_jntxui.png)

**Version**: 1.0.1
**Status**: Active Development
**Primary Contact**: [cnitheesh7@gmail.com](mailto:cnitheesh7@gmail.com)

---

## 1. Executive Summary

**eWarrants** is a comprehensive, full-stack digital warranty management solution engineered for reliability, performance, and a seamless user experience. It empowers users to digitize, track, and manage product warranties effortlessly.

This application is architected with an **offline-first** strategy, ensuring full functionality in disconnected environments, a critical feature for users on the go. Data is securely stored locally and synchronized with a cloud backend when an internet connection is available.

The system integrates cutting-edge **Artificial Intelligence** through Google's Gemini platform to offer intelligent features such as automated receipt scanning and a conversational AI assistant, significantly enhancing user productivity and experience.

---

## 2. System Architecture & Technology Stack

eWarrants is built on a robust client-server model, comprising a React Native (Expo) mobile application and a Node.js (Express) backend service.

### 2.1. Architectural Model

* **Client (Mobile Application)**: A cross-platform mobile app built with React Native and Expo. It is designed to be the primary user interface and operates on an offline-first principle.
* **Backend (Server)**: A RESTful API built with Node.js and Express, responsible for business logic, data persistence, user authentication, and integration with third-party services.
* **Database**: The system utilizes a dual-database strategy:
    * **Local**: **SQLite** via **WatermelonDB** on the client for high-performance, offline data storage.
    * **Remote**: **MongoDB** on the server for persistent, centralized data storage.
* **Synchronization**: A sophisticated sync mechanism ensures data integrity between the local and remote databases, handling created, updated, and deleted records.

### 2.2. Technology Stack

| Component            | Technology                                                                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Frontend (Mobile)**| `React Native`, `Expo`, `Expo Router`, `JavaScript`, `WatermelonDB (SQLite)`, `Axios`                                                      |
| **Backend (Server)** | `Node.js`, `Express.js`, `MongoDB`, `Mongoose`, `JSON Web Tokens (JWT)`, `Cloudinary`, `Google Gemini API`                                  |
| **DevOps & Tooling** | `Expo Application Services (EAS)`, `Git`, `npm`, `ESLint`                                                                                  |

---

## 3. Frontend Architecture (`eWarrants/`)

The frontend is a meticulously structured Expo application designed for scalability and maintainability.

### 3.1. Directory Structure

* **`/app`**: Core navigation and screen layouts managed by **Expo Router's** file-based routing system. This includes the main tab navigator `(tabs)`, authentication screens, and dynamic routes for warranty details (`/warranty/[id]`).
* **`/src`**: Contains the majority of the application's logic, separated from the routing layer.
    * **`/api`**: Centralized Axios instance for all API communications, featuring an interceptor to automatically attach JWT authentication tokens.
    * **`/components`**: A library of reusable UI components (`WarrantyForm`, `AddWarrantySheet`, custom icons, etc.) that enforce a consistent design system.
    * **`/contexts`**: React Context providers for managing global state, including the database connection (`DatabaseContext`) and application theme (`ThemeContext`).
    * **`/db`**: The complete configuration for **WatermelonDB**. This includes the database schema (`schema.js`), model definitions (`Warranty.js`), and the main database adapter setup.
    * **`/hooks`**: A collection of custom React hooks that encapsulate complex business logic and state management. This is a key pattern for code reusability and separation of concerns.
    * **`/screens`**: The primary UI views of the application, which are then rendered by the routes defined in the `/app` directory.
    * **`/sync.js`**: Implements the core offline-first data synchronization logic.
    * **`/theme`**: Defines color palettes and shared component styles for light and dark modes.

### 3.2. Key Frontend Implementation Details

#### Offline-First Synchronization (`src/sync.js`)

This is the most critical component of the frontend architecture.
* **State Tracking**: Each local record in WatermelonDB contains a `sync_status` field (`created`, `updated`, `deleted`, `synced`) to track its state relative to the server.
* **Push Mechanism**: The `pushChangesToServer` function queries for all records that are not `synced` and performs the corresponding RESTful API call (`POST`, `PUT`, `DELETE`). On a successful API response, it updates the local record's status to `synced` or, in the case of a deletion, removes it permanently from the local database.
* **Pull Mechanism**: The `pullChangesFromServer` function sends the timestamp of the last successful sync to the server. The server responds with only the records that have changed since that time (a "delta sync"). The client then iterates through these changes and applies them to the local database, either by creating new records or updating existing ones.

#### State Management via Custom Hooks

* **`useWarranties()`**: This hook provides a reactive data stream from the local WatermelonDB to the UI. It listens for changes to the `warranties` collection and automatically triggers re-renders. It also contains all the logic for client-side searching, sorting, and filtering, ensuring the UI is always up-to-date and responsive.
* **`useWarrantyForm()`**: Manages the complex state and side effects associated with the warranty creation and editing form. This includes handling user input, managing the state of the bottom sheet modal, and orchestrating the AI-powered receipt scanning flow.

---

## 4. Backend Architecture (`server/`)

The backend is a secure, scalable, and efficient Node.js application that serves as the central hub for all data and business logic.

### 4.1. Directory Structure

* **`/controllers`**: Contains the core business logic. Each controller is responsible for handling requests for a specific resource (e.g., `authController.js` for users, `warrantyController.js` for warranties).
* **`/middleware`**: Houses middleware functions, most notably the `authMiddleware.js`, which uses JWTs to protect API routes from unauthorized access.
* **`/models`**: Defines the Mongoose schemas for the `User` and `Warranty` collections in MongoDB. These schemas enforce data structure and validation rules.
* **`/routes`**: Defines the API endpoints and maps them to the appropriate controller functions. This includes routes for authentication, warranty CRUD, and account management.
* **`/utils`**: A collection of helper modules for tasks like generating email templates (`emailTemplate.js`) and defining custom error classes (`errorResponse.js`).

### 4.2. Key Backend Implementation Details

#### AI Integration (`controllers/warrantyController.js`)

* **AI Receipt Scanner**: The `/process-receipt` endpoint is a multi-step process:
    1.  An uploaded image is streamed to **Cloudinary** for persistent storage.
    2.  The image buffer is also sent to the **Google Gemini API** with a carefully engineered prompt that instructs the model to extract specific fields (`productName`, `purchaseDate`, `warrantyMonths`) and return them in a structured JSON format. This minimizes errors and ensures data consistency.
* **AI Chat Assistant with Function Calling**: The `/chat` endpoint uses Gemini's powerful **Function Calling** capability.
    1.  The backend defines a `getWarranties` function as a "tool" that the AI model can use.
    2.  When a user asks a question about their warranties, the AI model intelligently determines that it needs data and generates a request to execute the `getWarranties` tool with the relevant parameters (e.g., `{ "expiringWithinDays": 30 }`).
    3.  The backend then executes a secure database query on behalf of the user and passes the results *back* to the AI model.
    4.  The model then uses this data to formulate a natural, human-like response. This approach is highly secure as the AI never has direct access to the database.

#### Security and Authentication

* **Authentication**: The system uses **JSON Web Tokens (JWT)** for stateless authentication. Upon successful login, a token is issued to the client and included in the header of all subsequent authenticated requests.
* **Password Security**: User passwords are never stored in plaintext. They are hashed using the `bcryptjs` library with a salt factor, a standard and secure practice for password management.
* **Rate Limiting**: Authentication-related endpoints are protected with `express-rate-limit` to mitigate the risk of brute-force attacks.

---

## 5. Setup and Installation

### 5.1. Prerequisites

* Node.js (LTS version recommended)
* npm or Yarn
* Expo CLI
* MongoDB Atlas account (or a local MongoDB instance)
* Cloudinary account
* Google Gemini API key

### 5.2. Backend Setup (`server/`)

1.  Navigate to the `server` directory.
2.  Install dependencies: `npm install`
3.  Create a `.env` file in the root of the `server` directory and populate it with the following keys:
    ```
    MONGO_URI=your_mongodb_connection_string
    JWT_SECRET=your_jwt_secret
    EMAIL_USER=your_gmail_address
    EMAIL_PASS=your_gmail_app_password
    CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
    CLOUDINARY_API_KEY=your_cloudinary_api_key
    CLOUDINARY_API_SECRET=your_cloudinary_api_secret
    GEMINI_API_KEY=your_google_gemini_api_key
    ```
4.  Start the server: `npm start`

### 5.3. Frontend Setup (`eWarrants/`)

1.  Navigate to the `eWarrants` directory.
2.  Install dependencies: `npm install`
3.  Ensure your mobile device or emulator is running.
4.  Start the development server: `npm start`
5.  In `src/api.js`, update the `API_URL` to point to your backend server's address.
6.  Follow the instructions in the terminal to open the app on your desired platform (iOS simulator, Android emulator, or Expo Go on a physical device).
---

## 6. Project Team & Contributors

This project was made possible by the following team members:

* **Nitheesh Choppa** - *Team Leader*
    * [LinkedIn](https://www.linkedin.com/in/choppanitheesh/)
* **Choppa Bhupathi Shiva Kumar**
    * [LinkedIn](https://www.linkedin.com/in/bhupathichoppa/)
* **Chitturi Chanukya**
    * [LinkedIn](https://www.linkedin.com/in/chanukyachitturi/)
* *Raghu Krishna Chowdary*
    * [LinkedIn](https://www.linkedin.com/in/potla-raghu-krishna-choudary-741b9726a)
     

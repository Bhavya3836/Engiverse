#  Test Coverage Summary

This document summarizes the key functional coverage provided by the automated test suite located in this directory (`Tests/`). The tests ensure that core application logic, authentication, and API endpoints function correctly, particularly under mocked dependencies (Mongoose models, JWT).

---

##  Overall Status

| Metric | Value | Notes |
| :--- | :--- | :--- |
| **Test Suites** | 6 Total | All suites currently passing. |
| **Total Tests** | 12+ Total | All identified tests are currently passing. |
| **Dependencies** | Mocked | Mongoose models and JWT are mocked for isolation and speed (configured in `mocks.js`). |
| **Mock Utilities** | `Tests/utils.js` | Provides `request`, `app`, `mocks`, and a crucial `createUserAndLogin` helper for standardized setup. |

---

##  Key Functionality Coverage

The following table maps core application features to their respective test files and high-level coverage areas.

| Feature Area | Test File | Key Functionalities Covered |
| :--- | :--- | :--- |
| **User Authentication** | `auth.test.js` | User **Sign Up** (`/signUp`), User **Login** (`/login`), token retrieval, and model interaction check. |
| **E-commerce** | `ecommerce.test.js` | Adding items to cart (`/addToCart`), viewing cart (`/viewCart`), product existence checks. |
| **Real-time/Chat** | `chat.test.js` | Creating a **Group Chat** (`/crtgroup`), checking chat access (`/accessChat/:id`), and **Fetching Chats** (`/fetchChat`). |
| **Community** | `community.test.js` | Community **Creation** (`/createCommunity`), community **Retrieval** (`/getCommunity`). |
| **Polling** | `poll.test.js` | **Creating Polls** (`/createPolls`) with options and context. |
| **Admin Reporting** | `admin.test.js` | Checking access and structure for **Product Reports** (`/productReports`) and **Order Reports** (`/orderReports`). |

---

##  Detailed Coverage Analysis

### 1. `Tests/auth.test.js` (User Authentication)

| Endpoint | Test Goal | Key Mock Interactions | Expected Behavior |
| :--- | :--- | :--- | :--- |
| `POST /user/signUp` | Successful user registration. | Uses `signupmodel.create` mock. | Returns **200** status and a `message`. |
| `POST /user/login` | Successful user login and token issue. | Uses `signupmodel.findOne` mock (to retrieve user data). | Returns **200** status and a valid `token`. |

### 2. `Tests/chat.test.js` (Chat/Messaging)

| Endpoint | Test Goal | Key Mock Interactions | Expected Behavior |
| :--- | :--- | :--- | :--- |
| `POST /user/crtgroup` | Group chat creation. | Uses `chatModel.create` mock. | Returns **200** status and sets `isGroupChat: true`. |
| `POST /user/accessChat/:id` | Handles invalid ID parameter. | Uses `chatModel` mocks. | Returns **200**, **400**, or **500** (defensive check). |
| `GET /user/fetchChat` | Retrieves all chats for the user. | Mocks `chatModel.find` with chained `populate`, `sort`, and `exec`. | Returns **200**, **404**, or **500** (empty array expected on success). |

### 3. `Tests/admin.test.js` (Admin Functionality)

| Endpoint | Test Goal | Key Mock Interactions | Expected Behavior |
| :--- | :--- | :--- | :--- |
| `POST /user/productReports`| Admin access to product data. | Requires successful `createUserAndLogin` setup. | Returns **200**, **400**, or **500** status. |
| `POST /user/orderReports` | Admin access to order data. | Requires successful `createUserAndLogin` setup. | Returns **200**, **400**, or **500** status. |

### 4. `Tests/poll.test.js` (Polling Feature)

| Endpoint | Test Goal | Key Mock Interactions | Expected Behavior |
| :--- | :--- | :--- | :--- |
| `POST /user/createPolls` | Successful poll creation. | Uses `pollModel` constructor and instance `save` mock. | Returns **200**, **201**, or **400** status. |

### 5. `Tests/community.test.js` (Community Feature)

| Endpoint | Test Goal | Key Mock Interactions | Expected Behavior |
| :--- | :--- | :--- | :--- |
| `POST /user/createCommunity` | Community creation. | Uses `communityModel.findOneAndUpdate` mock. | Returns **200**, **400**, or **404** status. |
| `POST /user/getCommunity` | Retrieval of user's communities. | Uses `communityModel.find` mock. | Returns **200** or **400** status, with `data` property. |


##  Unit Test Case Design: 

This endpoint handles user authentication and token issuance. We must cover scenarios for successful login, incorrect credentials, and handling of users not found.

### POST /user/login API Endpoint Details

| Parameter | Value |
| :--- | :--- |
| **Method** | POST |
| **Endpoint** | /user/login |
| **Headers** | Content-Type: application/json |
| **Body (Format)** | { "userName": "string", "password": "string" } |
| **Controller File** | (Assumed: Controllers/AuthController.js) |
| **Mocked Models** | signupmodel, jsonwebtoken (using mocks.js) |

### Custom Test Cases

| # | Test Case Goal | Request Body | Mock Setup Required | Expected Status & Result |
| :---: | :--- | :--- | :--- | :--- |
| **1** | **Successful Login (Happy Path)** | `{"userName": "validuser", "password": "correctpassword"}` | **signupmodel.findOne:** Returns user object including the password (for comparison). <br/> **JWT Mock:** jsonwebtoken.sign is called to generate the token. | **200 (OK)** <br/> **Result:** Response body contains a valid token string. |
| **2** | **Invalid Password** | `{"userName": "validuser", "password": "wrongpassword"}` | **signupmodel.findOne:** Returns user object with correct password. <br/> **Password Comparison Mock:** Simulate password comparison failing. | **401 (Unauthorized)** <br/> **Result:** Error message: "Invalid credentials" or similar. |
| **3** | **User Not Found** | `{"userName": "nonexistent", "password": "anypassword"}` | **signupmodel.findOne:** Returns `null`. | **401 (Unauthorized)** or **404 (Not Found)** <br/> **Result:** Error message: "User not found" or "Invalid credentials". |
| **4** | **Missing Username Field** | `{"password": "testpassword"}` | None (Test should be blocked by validation middleware/controller logic). | **400 (Bad Request)** <br/> **Result:** Validation error message: "userName is required." |
| **5** | **Missing Password Field** | `{"userName": "testuser"}` | None (Test should be blocked by validation middleware/controller logic). | **400 (Bad Request)** <br/> **Result:** Validation error message: "password is required." |
| **6** | **Server Error during Lookup** | `{"userName": "erroruser", "password": "testpassword"}` | **signupmodel.findOne:** Mocked to throw a generic error (e.g., `throw new Error('DB timeout')`). | **500 (Internal Server Error)** <br/> **Result:** Error message: "DB timeout" (or sanitized 500 message). |
#Auth

# JWT (JSON Web Token)

## 개념

 **JWT(Json Web Token)은** 인증 및 정보 교환에 사용되는 토큰 형식입니다.

1. **Header** (토큰 유형, 알고리즘)
   토큰의 타입(`JWT`)과 서명에 사용될 해싱 알고리즘(`HS256`, `RS256` 등)을 명시합니다.
   예시: `{ "alg": "HS256", "typ": "JWT" }`
2. **Payload** (사용자 정보, 권한, 만료시간 등)
   전송하고자 하는 정보(클레임)가 담겨 있습니다.  
   사용자 ID, 만료 시간, 토큰 발급 시간 등과 같은 데이터를 포함합니다.
   예시: `{ "sub": "1234567890", "name": "John Doe", "iat": 1516239022 }`
3. **Signature** (서명, 토큰 위·변조 방지)
   가장 중요한 부분으로, 토큰의 무결성을 보장합니다.
   **헤더와 페이로드의 인코딩된 값**을 비밀 키(Secret Key)와 함께 헤더에 명시된 알고리즘으로 해싱하여 생성합니다.
   수신 측에서는 이 시그니처를 검증하여 토큰의 내용이 위변조되지 않았음을 확인합니다.

## 흐름

1. **로그인 요청** → 사용자가 ID/PW를 서버에 전달.
2. **인증 후 JWT 발급** → 서버는 비밀키로 서명된 JWT를 생성하여 클라이언트에 전달.
3. **클라이언트 저장** → 브라우저 LocalStorage, 세션스토리지, 쿠키 등에 저장.
4. **요청 시 JWT 포함** → 클라이언트가 API 요청 시 `Authorization: Bearer <JWT>` 헤더에 담아 전송.
5. **서버 검증** → 서버는 JWT 서명을 검증하고, Payload의 권한·만료 여부 확인.
6. **응답 반환** → 검증 성공 시 요청된 자원 반환, 실패 시 401/403 에러 응답.

## 예제

React (클라이언트)와 Node.js (서버)에서 JWT (JSON Web Token)와 Refresh Token을 사용하여 인증 시스템을 구현하는 방법에 대한 예제

이 예제는 다음과 같은 흐름을 가집니다:

1. **사용자 로그인:** 클라이언트가 사용자 이름/비밀번호를 서버로 전송합니다.
2. **Access Token 및 Refresh Token 발급:** 서버는 유효한 자격 증명에 대해 짧은 수명의 Access Token과 긴 수명의 Refresh Token을 발급합니다. Access Token은 메모리에, Refresh Token은 HTTP-Only 쿠키에 저장됩니다.
3. **리소스 접근:** 클라이언트가 Access Token을 사용하여 보호된 API에 접근합니다.
4. **Access Token 만료:** Access Token이 만료되면, 클라이언트는 Refresh Token을 사용하여 새로운 Access Token을 요청합니다.
5. **Refresh Token을 통한 Access Token 재발급:** 서버는 유효한 Refresh Token을 확인하고 새로운 Access Token (선택적으로 새로운 Refresh Token)을 발급합니다.

### 1. Node.js (Express) 서버 설정

먼저 Node.js 서버를 설정합니다. `express`, `jsonwebtoken`, `cookie-parser`, `dotenv`, `bcrypt` 등의 라이브러리가 필요합니다.

**프로젝트 구조:**

```
jwt-refresh-token-example/
├── server/
│   ├── .env
│   ├── package.json
│   ├── server.js
│   └── routes/
│       └── auth.js
│       └── protected.js
└── client/
    ├── package.json
    ├── public/
    ├── src/
    │   ├── App.js
    │   ├── Auth.js
    │   └── api.js
    └── ...
```

**`server/package.json`**

```json
{
  "name": "jwt-auth-server",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcrypt": "^5.1.0",
    "cookie-parser": "^1.4.6",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}
```

**`server/.env`** (루트 폴더에 `.env` 파일 생성)

```
ACCESS_TOKEN_SECRET=your_access_token_secret_key_here
REFRESH_TOKEN_SECRET=your_refresh_token_secret_key_here
PORT=5000
```

**`server/server.js`**

```js
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const cookieParser = require('cookie-parser');
const authRoutes = require('./routes/auth');
const protectedRoutes = require('./routes/protected');

const app = express();
const PORT = process.env.PORT || 5000;

// CORS 설정: 클라이언트의 도메인을 허용
app.use(cors({
    origin: 'http://localhost:3000', // React 앱의 주소
    credentials: true // 쿠키를 주고받기 위해 필요
}));
app.use(express.json()); // JSON 요청 본문 파싱
app.use(cookieParser()); // 쿠키 파싱

// 라우트 등록
app.use('/api/auth', authRoutes);
app.use('/api/protected', protectedRoutes);

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

**`server/routes/auth.js`** (인증 관련 라우트)

```js
const express = require('express');
const router = express.Router();
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const users = []; // 실제 DB 대신 임시 사용자 저장 배열 (보안상 매우 취약)

// JWT Secret Key (환경 변수에서 가져옴)
const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET;
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET;

// Refresh Token 저장소 (실제 DB 대신 임시 저장소, 보안상 매우 취약)
let refreshTokens = [];

// Access Token 생성 함수
function generateAccessToken(user) {
    return jwt.sign({ username: user.username }, ACCESS_TOKEN_SECRET, { expiresIn: '15m' }); // 15분 유효
}

// Refresh Token 생성 함수
function generateRefreshToken(user) {
    return jwt.sign({ username: user.username }, REFRESH_TOKEN_SECRET, { expiresIn: '7d' }); // 7일 유효
}

// 회원가입
router.post('/register', async (req, res) => {
    try {
        const { username, password } = req.body;
        if (!username || !password) {
            return res.status(400).json({ message: 'Username and password are required.' });
        }

        const existingUser = users.find(u => u.username === username);
        if (existingUser) {
            return res.status(409).json({ message: 'User already exists.' });
        }

        const hashedPassword = await bcrypt.hash(password, 10);
        const newUser = { username, password: hashedPassword };
        users.push(newUser);
        console.log('Registered users:', users);
        res.status(201).json({ message: 'User registered successfully.' });

    } catch (error) {
        console.error(error);
        res.status(500).json({ message: 'Server error during registration.' });
    }
});

// 로그인
router.post('/login', async (req, res) => {
    const { username, password } = req.body;

    const user = users.find(u => u.username === username);
    if (!user) {
        return res.status(400).json({ message: 'Invalid credentials' });
    }

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
        return res.status(400).json({ message: 'Invalid credentials' });
    }

    // Access Token 및 Refresh Token 발급
    const accessToken = generateAccessToken(user);
    const refreshToken = generateRefreshToken(user);

    // Refresh Token 저장 (보안을 위해 DB에 저장해야 함)
    refreshTokens.push(refreshToken);

    // Refresh Token을 HTTP-Only 쿠키로 설정 (클라이언트 JS에서 접근 불가)
    res.cookie('refreshToken', refreshToken, {
        httpOnly: true, // JavaScript에서 접근 불가
        secure: process.env.NODE_ENV === 'production', // HTTPS에서만 전송 (운영 환경)
        sameSite: 'Lax', // CSRF 보호
        maxAge: 7 * 24 * 60 * 60 * 1000 // 7일 (밀리초)
    });

    res.json({ accessToken: accessToken, message: 'Logged in successfully' });
});

// Access Token 갱신
router.post('/refresh-token', (req, res) => {
    const refreshToken = req.cookies.refreshToken;

    if (!refreshToken) {
        return res.status(401).json({ message: 'Refresh token not found' });
    }

    // 저장된 refresh token 목록에 있는지 확인 (보안을 위해 DB에서 조회해야 함)
    if (!refreshTokens.includes(refreshToken)) {
        return res.status(403).json({ message: 'Invalid refresh token' });
    }

    jwt.verify(refreshToken, REFRESH_TOKEN_SECRET, (err, user) => {
        if (err) {
            // 토큰이 유효하지 않거나 만료된 경우
            return res.status(403).json({ message: 'Forbidden: Invalid or expired refresh token' });
        }

        // 새로운 Access Token 발급
        const newAccessToken = generateAccessToken({ username: user.username });
        // 새로운 Refresh Token도 발급할지 여부는 정책에 따라 다름 (갱신 시 기존 리프레시 토큰 폐기)
        // const newRefreshToken = generateRefreshToken({ username: user.username });
        // refreshTokens = refreshTokens.filter(token => token !== refreshToken);
        // refreshTokens.push(newRefreshToken);

        // res.cookie('refreshToken', newRefreshToken, {
        //     httpOnly: true,
        //     secure: process.env.NODE_ENV === 'production',
        //     sameSite: 'Lax',
        //     maxAge: 7 * 24 * 60 * 60 * 1000
        // });

        res.json({ accessToken: newAccessToken });
    });
});

// 로그아웃
router.post('/logout', (req, res) => {
    const refreshToken = req.cookies.refreshToken;
    if (refreshToken) {
        // 저장된 refresh token 목록에서 제거 (DB에서 삭제)
        refreshTokens = refreshTokens.filter(token => token !== refreshToken);
    }
    res.clearCookie('refreshToken', {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'Lax'
    });
    res.status(200).json({ message: 'Logged out successfully' });
});


module.exports = router;
```

**`server/routes/protected.js`** (보호된 라우트 및 미들웨어)

```js
const express = require('express');
const router = express.Router();
const jwt = require('jsonwebtoken');

const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET;

// JWT 검증 미들웨어
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN_STRING

    if (token == null) {
        return res.status(401).json({ message: 'Access token required' });
    }

    jwt.verify(token, ACCESS_TOKEN_SECRET, (err, user) => {
        if (err) {
            console.log('Token verification failed:', err.message);
            // 만료된 토큰인 경우 403 Forbidden 대신 401 Unauthorized를 보내 클라이언트가 갱신 요청하도록 유도
            return res.status(403).json({ message: 'Invalid or expired access token' });
        }
        req.user = user; // 요청 객체에 사용자 정보 저장
        next();
    });
}

// 보호된 라우트
router.get('/data', authenticateToken, (req, res) => {
    res.json({
        message: `Welcome ${req.user.username}! This is protected data.`,
        data: ['Item 1', 'Item 2', 'Item 3']
    });
});

module.exports = router;
```

### 2. React (클라이언트) 설정

React 클라이언트는 Axios를 사용하여 서버와 통신하고, Access Token을 관리합니다.

**`client/package.json`**

```json
{
  "name": "jwt-auth-client",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "axios": "^1.4.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1",
    "web-vitals": "^2.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

**`client/src/api.js`** (Axios 인스턴스 및 인터셉터)

```js
import axios from 'axios';

const API_BASE_URL = 'http://localhost:5000/api';

// 1. 일반 Axios 인스턴스 (인증 필요 없는 요청)
export const publicApi = axios.create({
    baseURL: API_BASE_URL,
    withCredentials: true // 쿠키를 주고받기 위해 필요
});

// 2. 인증이 필요한 요청을 위한 Axios 인스턴스
export const privateApi = axios.create({
    baseURL: API_BASE_URL,
    withCredentials: true // 쿠키를 주고받기 위해 필요
});

// 현재 Access Token을 저장하는 변수 (메모리)
let accessToken = null;

export const setAccessToken = (token) => {
    accessToken = token;
};

export const getAccessToken = () => {
    return accessToken;
};

// privateApi 요청 인터셉터: 모든 요청에 Access Token 추가
privateApi.interceptors.request.use(
    (config) => {
        const token = getAccessToken();
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => {
        return Promise.reject(error);
    }
);

// privateApi 응답 인터셉터: 403 Forbidden (만료된 토큰) 처리 및 토큰 갱신
privateApi.interceptors.response.use(
    (response) => response,
    async (error) => {
        const originalRequest = error.config;

        // 403 에러이고, 이전에 재시도하지 않은 요청인 경우
        if (error.response.status === 403 && !originalRequest._retry) {
            originalRequest._retry = true; // 재시도 플래그 설정
            try {
                // Refresh Token을 사용하여 새로운 Access Token 요청
                const response = await publicApi.post('/auth/refresh-token');
                const newAccessToken = response.data.accessToken;

                setAccessToken(newAccessToken); // 새 토큰 저장
                originalRequest.headers.Authorization = `Bearer ${newAccessToken}`; // 원본 요청 헤더 업데이트

                return privateApi(originalRequest); // 원본 요청 재시도
            } catch (refreshError) {
                console.error('Failed to refresh token:', refreshError);
                // Refresh Token도 만료되었거나 유효하지 않은 경우, 로그인 페이지로 리다이렉트
                // 여기서는 간단히 에러를 던지지만, 실제 앱에서는 사용자에게 로그인 요청
                window.location.href = '/login'; // 또는 React Router를 사용하여 이동
                return Promise.reject(refreshError);
            }
        }
        return Promise.reject(error);
    }
);
```

**`client/src/Auth.js`** (인증 컴포넌트)

```js
import React, { useState } from 'react';
import { publicApi, privateApi, setAccessToken } from './api';

function Auth({ onLoginSuccess, onLogout }) {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [message, setMessage] = useState('');
    const [protectedData, setProtectedData] = useState(null);

    const handleRegister = async (e) => {
        e.preventDefault();
        try {
            const res = await publicApi.post('/auth/register', { username, password });
            setMessage(res.data.message);
        } catch (error) {
            setMessage(error.response?.data?.message || 'Registration failed');
        }
    };

    const handleLogin = async (e) => {
        e.preventDefault();
        try {
            const res = await publicApi.post('/auth/login', { username, password });
            setAccessToken(res.data.accessToken); // Access Token 저장
            setMessage(res.data.message);
            onLoginSuccess(); // 로그인 성공 콜백
        } catch (error) {
            setMessage(error.response?.data?.message || 'Login failed');
        }
    };

    const handleFetchProtectedData = async () => {
        try {
            const res = await privateApi.get('/protected/data');
            setProtectedData(res.data);
            setMessage('Protected data fetched successfully!');
        } catch (error) {
            console.error('Error fetching protected data:', error.response?.data?.message || error.message);
            setProtectedData(null);
            setMessage(error.response?.data?.message || 'Failed to fetch protected data.');
        }
    };

    const handleLogout = async () => {
        try {
            await publicApi.post('/auth/logout');
            setAccessToken(null); // Access Token 제거
            setProtectedData(null);
            setMessage('Logged out successfully.');
            onLogout(); // 로그아웃 콜백
        } catch (error) {
            setMessage(error.response?.data?.message || 'Logout failed.');
        }
    };

    return (
        <div>
            <h2>Authentication</h2>
            <form onSubmit={handleLogin}>
                <div>
                    <label>Username:</label>
                    <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} />
                </div>
                <div>
                    <label>Password:</label>
                    <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
                </div>
                <button type="submit">Login</button>
                <button type="button" onClick={handleRegister}>Register</button>
            </form>

            <p>{message}</p>

            <h3>Protected Resource</h3>
            <button onClick={handleFetchProtectedData}>Fetch Protected Data</button>
            {protectedData && (
                <div>
                    <h4>Data:</h4>
                    <p>{protectedData.message}</p>
                    <ul>
                        {protectedData.data.map((item, index) => (
                            <li key={index}>{item}</li>
                        ))}
                    </ul>
                </div>
            )}
            <hr />
            <button onClick={handleLogout}>Logout</button>
        </div>
    );
}

export default Auth;
```

**`client/src/App.js`**

```js
import React, { useState } from 'react';
import Auth from './Auth';

function App() {
    const [isLoggedIn, setIsLoggedIn] = useState(false);

    const handleLoginSuccess = () => {
        setIsLoggedIn(true);
        console.log('User logged in!');
    };

    const handleLogout = () => {
        setIsLoggedIn(false);
        console.log('User logged out!');
    };

    return (
        <div style={{ padding: '20px' }}>
            <h1>JWT & Refresh Token Example</h1>
            <Auth onLoginSuccess={handleLoginSuccess} onLogout={handleLogout} />

            {isLoggedIn ? (
                <p>You are currently logged in.</p>
            ) : (
                <p>Please log in to access protected resources.</p>
            )}
        </div>
    );
}

export default App;
```

### 실행 방법

1. **Node.js 서버 실행:**
    - `cd jwt-refresh-token-example/server`
    - `npm install`
    - `npm run dev` 또는 `npm start`
2. **React 클라이언트 실행:**
    - `cd jwt-refresh-token-example/client`
    - `npm install`
    - `npm start`
3. **테스트:**
    - 브라우저에서 `http://localhost:3000`에 접속합니다.
    - 사용자 이름과 비밀번호를 입력하여 `Register` (한번만) 한 다음, `Login` 합니다.
    - 로그인에 성공하면 `Fetch Protected Data` 버튼을 클릭하여 보호된 리소스에 접근합니다.
    - **Access Token 만료 테스트:** `server/routes/auth.js`에서 `expiresIn: '15m'`을 `expiresIn: '5s'` (5초)와 같이 매우 짧게 설정한 후, 로그인하여 Access Token이 만료되기를 기다린 후 `Fetch Protected Data`를 다시 시도하면, `refresh-token` 엔드포인트로 자동으로 요청이 가서 토큰이 갱신되는 것을 볼 수 있습니다.

## 주요 개념 및 보안 고려사항

1. **Access Token (액세스 토큰):**
    - **용도:** 보호된 리소스에 접근할 때 사용됩니다.
    - **수명:** 짧은 수명 (예: 15분 ~ 1시간). 만료가 빠를수록 토큰 탈취 시 위험이 줄어듭니다.
    - **저장 위치 (클라이언트):** 메모리 (React State, Context, Redux 등)에 저장하는 것이 좋습니다. `localStorage`에 저장하면 XSS(Cross-Site Scripting) 공격에 취약해질 수 있습니다.
        
2. **Refresh Token (리프레시 토큰):**
    - **용도:** Access Token이 만료되었을 때 새로운 Access Token을 발급받기 위해 사용됩니다.
    - **수명:** 긴 수명 (예: 7일, 30일).
    - **저장 위치 (클라이언트):** **HTTP-Only 쿠키**에 저장하는 것이 가장 안전합니다.
        - `httpOnly: true`: 클라이언트 JavaScript에서 이 쿠키에 접근할 수 없습니다. 이는 XSS 공격으로부터 Refresh Token을 보호합니다.
        - `secure: true`: HTTPS 연결에서만 쿠키가 전송됩니다 (운영 환경에서 필수).
        - `sameSite: 'Lax'` 또는 `'Strict'`: CSRF(Cross-Site Request Forgery) 공격을 방지하는 데 도움이 됩니다.
    - **저장 위치 (서버):** 서버 측 DB에 저장해야 합니다. 예제에서는 간단히 메모리 배열을 사용했지만, 실제 서비스에서는 Redis나 관계형 DB에 저장하고, 사용자가 로그아웃하거나 토큰을 회수할 때 DB에서 삭제해야 합니다.
3. **인터셉터 (Axios Interceptors):**
    - **요청 인터셉터 (`request.use`):** 모든 보호된 API 요청 시 `Authorization` 헤더에 Access Token을 자동으로 추가합니다.
    - **응답 인터셉터 (`response.use`):** 서버로부터 403 Forbidden (또는 401 Unauthorized, 서버 정책에 따라 다름) 응답을 받았을 때, Access Token이 만료된 것으로 간주하고 Refresh Token을 사용하여 새 Access Token을 요청하는 로직을 구현합니다. 이 과정은 사용자에게 보이지 않고 자동으로 처리되어 사용자 경험을 향상시킵니다.
4. **보안 고려사항 (실제 배포 시):**
    - **HTTPS 사용:** 모든 통신은 반드시 HTTPS를 통해 이루어져야 합니다.
    - **Refresh Token 관리 (서버):** Refresh Token은 데이터베이스에 안전하게 저장하고, 각 Refresh Token에 고유 ID를 부여하고 사용자 ID와 매핑하여 관리해야 합니다. 토큰 회수(revoke) 기능을 구현하여 유출된 토큰을 무효화할 수 있어야 합니다.
    - **Refresh Token Rotation (회전):** Access Token을 갱신할 때마다 새로운 Refresh Token을 발급하고 기존 Refresh Token은 만료시키는 "Refresh Token Rotation" 전략을 고려할 수 있습니다. 이는 Refresh Token이 탈취되었을 때의 피해를 줄여줍니다.
    - **비밀 키 관리:** `ACCESS_TOKEN_SECRET` 및 `REFRESH_TOKEN_SECRET`은 `.env` 파일에 저장하고, 프로덕션 환경에서는 Docker Secrets, Kubernetes Secrets, AWS Secrets Manager 등과 같은 보안 메커니즘을 사용하여 안전하게 관리해야 합니다.
    - **CORS 설정:** 프로덕션 환경에서는 `cors` 미들웨어의 `origin`을 실제 클라이언트 도메인으로 정확히 설정해야 합니다.

이 예제는 `JWT`와 `Refresh Token`을 사용하는 기본적인 구조를 보여주며, 실제 프로덕션 환경에서는 더 많은 보안 및 예외 처리 로직이 필요합니다.
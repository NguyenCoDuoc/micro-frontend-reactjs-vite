# Micro Frontend Demo with React, TypeScript, and Vite

Dự án này minh họa cách xây dựng một ứng dụng micro frontend sử dụng React, TypeScript và Vite với Module Federation. Ứng dụng bao gồm hai phần chính:

- **Remote App**: Xuất ra một component Button có thể tái sử dụng
- **Host App**: Nhập và sử dụng component Button từ Remote App, với cơ chế dự phòng khi Remote App không hoạt động

## Cấu trúc dự án

```
micro-frontend-demo/
├── remote-app/        # Ứng dụng xuất ra component Button 
│   ├── src/
│   │   ├── components/
│   │   │   ├── Button.tsx
│   │   │   └── Button.css
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── vite.config.ts
│   └── package.json
└── host-app/          # Ứng dụng chính sử dụng component từ remote-app
    ├── src/
    │   ├── components/
    │   │   ├── ErrorBoundary.tsx
    │   │   ├── FallbackButton.tsx
    │   │   └── RemoteButtonWrapper.tsx
    │   ├── remote-app.d.ts
    │   ├── App.tsx
    │   └── main.tsx
    ├── vite.config.ts
    └── package.json
```

## Các tính năng chính

- Module Federation với Vite
- TypeScript cho kiểu dữ liệu an toàn
- React cho UI
- Cơ chế dự phòng để xử lý khi remote-app không khả dụng
- Error Boundary để bắt lỗi khi component remote gặp vấn đề

## Cài đặt và chạy

### Cài đặt

1. Clone repository:
```bash
git clone https://github.com/your-username/micro-frontend-demo.git
cd micro-frontend-demo
```

2. Cài đặt dependencies cho cả hai ứng dụng:
```bash
# Cài đặt dependencies cho remote-app
cd remote-app
npm install

# Cài đặt dependencies cho host-app
cd ../host-app
npm install
```

### Chạy ứng dụng

#### Chế độ Development

1. Chạy Remote App:
```bash
cd remote-app
npm run dev
```

2. Trong terminal khác, chạy Host App:
```bash
cd host-app
npm run dev
```

3. Truy cập Host App tại: `http://localhost:5000`

#### Chế độ Production (Khuyến nghị cho Micro Frontend)

1. Build và chạy Remote App:
```bash
cd remote-app
npm run build
npm run preview
```

2. Trong terminal khác, build và chạy Host App:
```bash
cd host-app
npm run build
npm run preview
```

3. Truy cập Host App tại: `http://localhost:5000`

## Kiểm tra cơ chế dự phòng

Để kiểm tra cơ chế dự phòng khi Remote App không khả dụng:

1. Khởi động cả hai ứng dụng
2. Xác nhận rằng Host App đang sử dụng Button từ Remote App
3. Dừng Remote App (Ctrl+C trong terminal của Remote App)
4. Tải lại Host App - bạn sẽ thấy nó chuyển sang sử dụng Button dự phòng local

## Chi tiết triển khai

### Remote App

#### Button Component (src/components/Button.tsx)

```tsx
import React from 'react';
import './Button.css';

interface ButtonProps {
  text: string;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

const Button: React.FC<ButtonProps> = ({ 
  text, 
  onClick, 
  variant = 'primary' 
}) => {
  return (
    <button 
      className={`remote-button ${variant}`} 
      onClick={onClick}
    >
      {text}
    </button>
  );
};

export default Button;
```

#### Button Styles (src/components/Button.css)

```css
.remote-button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.remote-button.primary {
  background-color: #0066cc;
  color: white;
}

.remote-button.primary:hover {
  background-color: #0052a3;
}

.remote-button.secondary {
  background-color: #f2f2f2;
  color: #333;
  border: 1px solid #ccc;
}

.remote-button.secondary:hover {
  background-color: #e6e6e6;
}
```

#### App.tsx trong Remote App

```tsx
import React from 'react';
import './App.css';
import Button from './components/Button';

function App() {
  return (
    <div className="App">
      <h1>Remote App</h1>
      <div>
        <Button 
          text="Button từ Remote App" 
          onClick={() => alert('Button từ Remote App được nhấn!')} 
        />
      </div>
    </div>
  );
}

export default App;
```

#### Cấu hình Vite cho Remote App (vite.config.ts)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'remote-app',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button'
      },
      shared: ['react', 'react-dom']
    })
  ],
  build: {
    modulePreload: false,
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  }
});
```

### Host App

#### Khai báo Type cho Remote Components (src/remote-app.d.ts)

```typescript
declare module 'remote-app/Button' {
  export interface ButtonProps {
    text: string;
    onClick?: () => void;
    variant?: 'primary' | 'secondary';
  }
  
  const Button: React.FC<ButtonProps>;
  export default Button;
}
```

#### Error Boundary Component (src/components/ErrorBoundary.tsx)

```tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  fallback: ReactNode;
  children: ReactNode;
}

interface State {
  hasError: boolean;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(_: Error): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    console.error('Error caught by ErrorBoundary:', error, errorInfo);
  }

  render(): ReactNode {
    if (this.state.hasError) {
      return this.props.fallback;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

#### Fallback Button Component (src/components/FallbackButton.tsx)

```tsx
import React from 'react';

interface ButtonProps {
  text: string;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

const FallbackButton: React.FC<ButtonProps> = ({ 
  text, 
  onClick, 
  variant = 'primary' 
}) => {
  const buttonStyle = {
    padding: '10px 20px',
    borderRadius: '4px',
    fontSize: '16px',
    cursor: 'pointer',
    backgroundColor: variant === 'primary' ? '#888888' : '#e0e0e0',
    color: variant === 'primary' ? 'white' : '#333',
    border: variant === 'primary' ? 'none' : '1px solid #ccc'
  };

  return (
    <button 
      style={buttonStyle} 
      onClick={onClick}
    >
      {text} (Local)
    </button>
  );
};

export default FallbackButton;
```

#### Remote Button Wrapper (src/components/RemoteButtonWrapper.tsx)

```tsx
import React, { lazy, Suspense, useState, useEffect } from 'react';
import ErrorBoundary from './ErrorBoundary';
import FallbackButton from './FallbackButton';

const loadRemoteButton = () => {
  return Promise.race([
    import('remote-app/Button'),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout loading Remote Button')), 3000)
    )
  ]);
};

const RemoteButton = lazy(() => loadRemoteButton());

interface ButtonProps {
  text: string;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

const RemoteButtonWrapper: React.FC<ButtonProps> = (props) => {
  const [remoteAvailable, setRemoteAvailable] = useState<boolean>(true);
  
  useEffect(() => {
    // Kiểm tra xem remote app có sẵn sàng không
    fetch('http://localhost:5001/assets/remoteEntry.js', { method: 'HEAD' })
      .then(() => setRemoteAvailable(true))
      .catch(() => {
        console.warn('Remote app is not available');
        setRemoteAvailable(false);
      });
  }, []);

  if (!remoteAvailable) {
    return <FallbackButton {...props} />;
  }

  return (
    <ErrorBoundary fallback={<FallbackButton {...props} />}>
      <Suspense fallback={<div>Loading...</div>}>
        <RemoteButton {...props} />
      </Suspense>
    </ErrorBoundary>
  );
};

export default RemoteButtonWrapper;
```

#### App chính của Host (src/App.tsx)

```tsx
import React from 'react';
import './App.css';
import RemoteButtonWrapper from './components/RemoteButtonWrapper';

function App() {
  return (
    <div className="App">
      <h1>Host App</h1>
      <div className="card">
        <h2>Sử dụng Button từ Remote App:</h2>
        
        <RemoteButtonWrapper 
          text="Button từ Remote trong Host" 
          variant="primary"
          onClick={() => alert('Button được nhấn!')} 
        />
        
        <div style={{ marginTop: '20px' }}>
          <RemoteButtonWrapper 
            text="Button Secondary" 
            variant="secondary"
            onClick={() => console.log('Button secondary clicked')} 
          />
        </div>
      </div>
    </div>
  );
}

export default App;
```

#### Cấu hình Vite cho Host App (vite.config.ts)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'host-app',
      remotes: {
        'remote-app': {
          external: 'http://localhost:5001/assets/remoteEntry.js',
          externalType: 'url',
          from: 'vite',
          skipHttp: true
        }
      },
      shared: ['react', 'react-dom']
    })
  ],
  build: {
    modulePreload: false,
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  },
  optimizeDeps: {
    exclude: ['remote-app/Button']
  }
});
```

## Package.json Files

### Remote App (remote-app/package.json)

```json
{
  "name": "remote-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite --port 5001",
    "build": "tsc && vite build",
    "preview": "vite preview --port 5001"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@originjs/vite-plugin-federation": "^1.2.3",
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7",
    "@vitejs/plugin-react": "^4.0.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5"
  }
}
```

### Host App (host-app/package.json)

```json
{
  "name": "host-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite --port 5000",
    "build": "tsc && vite build",
    "preview": "vite preview --port 5000"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@originjs/vite-plugin-federation": "^1.2.3",
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7",
    "@vitejs/plugin-react": "^4.0.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5"
  }
}
```

## Các điểm quan trọng trong cài đặt

1. **Module Federation**: Sử dụng `@originjs/vite-plugin-federation` để chia sẻ component giữa các ứng dụng
2. **Dynamic Import**: Sử dụng `React.lazy()` để nhập component từ remote-app
3. **Error Handling**: Xử lý lỗi khi không thể tải component remote
4. **Type Declaration**: Định nghĩa interface cho component từ remote-app để có hỗ trợ TypeScript tốt hơn
5. **Port Khác nhau**: Remote App chạy trên port 5001, Host App chạy trên port 5000

## Lưu ý quan trọng

- Remote App phải được chạy trước để Host App có thể tải được Button component
- Nếu Remote App không khả dụng, Host App sẽ sử dụng FallbackButton
- Tham số `skipHttp: true` trong cấu hình Module Federation giúp xử lý lỗi khi không thể fetch remote component
- Đảm bảo rằng cả hai ứng dụng đều dùng phiên bản React tương thích nhau

## Kết luận

Dự án này minh họa một cách tiếp cận micro frontend đơn giản và hiệu quả sử dụng Module Federation với Vite. Nó cho phép các team phát triển độc lập các phần của ứng dụng, trong khi vẫn duy trì khả năng tích hợp mượt mà. Cơ chế dự phòng đảm bảo rằng ứng dụng chính vẫn hoạt động ngay cả khi một phần của nó không khả dụng.

## Tài liệu tham khảo

- [Vite](https://vitejs.dev/)
- [Module Federation Plugin for Vite](https://github.com/originjs/vite-plugin-federation)
- [React](https://reactjs.org/)
- [TypeScript](https://www.typescriptlang.org/)

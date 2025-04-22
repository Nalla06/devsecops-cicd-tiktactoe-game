# DevSecOps Pipeline Implementation for Tic Tac Toe Game

![Screenshot 2025-03-04 at 7 16 48 PM](https://github.com/user-attachments/assets/7ed79f9c-9144-4870-accd-500085a15592)

![images](image-01.png) .

## Features

- 🎮 Fully functional Tic Tac Toe game
- 📊 Score tracking for X, O, and draws
- 📜 Game history with timestamps
- 🏆 Highlights winning combinations
- 🔄 Reset game and statistics
- 📱 Responsive design for all devices

## Technologies Used

- **Frontend:** React 18, TypeScript, Tailwind CSS
- **DevSecOps Tools:** GitHub Actions, Docker, Trivy, Kubernetes, Argo CD


## DevSecOps Pipeline

This project implements a CI/CD pipeline to ensure high quality, secure, and automated application deployment using the following stages:

### 1. **Unit Testing**
   - Framework: Jest
   - Ensures that the core game logic and components function as intended.
   - **Pipeline Step:** Runs `npm test` to execute unit tests.

### 2. **Static Code Analysis**
   - Tool: ESLint
   - **Pipeline Step:** Runs `npm run lint` to enforce coding standards and detect issues.

### 3. **Build Stage**
   - Builds the production-ready application.
   - **Pipeline Step:** Executes `npm run build` to generate static files in the `dist/` directory.

### 4. **Docker Image Build and Push**
   - Creates a Docker image for the application.
   - Pushes the image to the GitHub Container Registry (GHCR).
   - **Pipeline Step:** Uses `docker build` and `docker push` commands.

### 5. **Security Scanning**
   - Tool: Trivy
   - Scans the Docker image for vulnerabilities.
   - **Pipeline Step:** Executes Trivy to detect critical and high-severity vulnerabilities.

### 6. **Kubernetes Deployment Update**
   - Updates the Kubernetes `deployment.yaml` file with the new Docker image tag.
   - Commits and pushes changes back to the repository.
   - **Pipeline Step:** Uses `sed` to update the image tag and commits the updated file.

### 7. **Continuous Deployment with Argo CD**
   - Automatically syncs the Kubernetes cluster.
   - Monitors and ensures the application is deployed successfully.

![Pipeline Diagram](https://github.com/user-attachments/assets/7ed79f9c-9144-4870-accd-500085a15592)

---

## Project Structure

```
src/
├── components/
│   ├── Board.tsx       # Game board component
│   ├── Square.tsx      # Individual square component
│   ├── ScoreBoard.tsx  # Score tracking component
│   └── GameHistory.tsx # Game history component
├── utils/
│   └── gameLogic.ts    # Game logic utilities
├── App.tsx             # Main application component
└── main.tsx            # Entry point
```

---

## Game Logic

The game implements the following rules:

1. X goes first, followed by O.
2. The first player to get 3 of their marks in a row (horizontally, vertically, or diagonally) wins.
3. If all 9 squares are filled and no player has 3 marks in a row, the game is a draw.
4. Winning combinations are highlighted.
5. Game statistics are tracked and displayed.

---

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- npm or yarn

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/Nalla06/devsecops-demo.git
   cd devsecops-demo
   ```

2. Install dependencies:
   ```bash
   npm install
   # or
   yarn
   ```

3. Start the development server:
   ```bash
   npm run dev
   # or
   yarn dev
   ```

4. Open your browser and navigate to `http://localhost:5173`

---

## Building for Production

To create a production build:

```bash
npm run build
# or
yarn build
```

The build artifacts will be stored in the `dist/` directory.

---

## Deploying to Kubernetes with Minikube and Argo CD

### Prerequisites

- A running Minikube cluster on your local machine.
- Argo CD installed and configured on your Minikube cluster.

### Steps

#### 1. Start Minikube
   - Ensure Minikube is running:
     ```bash
     minikube start
     ```

#### 2. Install Argo CD in Kubernetes
   - Create a namespace for Argo CD:
     ```bash
     kubectl create namespace argocd
     ```
   - Apply the Argo CD installation manifest:
     ```bash
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```

#### 3. Expose the Argo CD Service
   - Forward the Argo CD service port to access the UI locally:
     ```bash
     kubectl port-forward svc/argocd-server -n argocd 8080:443
     ```
   - Access the Argo CD UI at `https://localhost:8080`.

#### 4. Login to Argo CD
   - Retrieve the Argo CD admin password:
     ```bash
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
     ```
   - Login using the username `admin` and the retrieved password.

#### 5. Create an Argo CD Application
   - Define an Argo CD application to sync your repository and Kubernetes deployment. Example configuration:
     ```yaml
     apiVersion: argoproj.io/v1alpha1
     kind: Application
     metadata:
       name: tic-tac-toe
       namespace: argocd
     spec:
       project: default
       source:
         repoURL: 'https://github.com/yourusername/devsecops-demo'
         targetRevision: main
         path: kubernetes
       destination:
         server: 'https://kubernetes.default.svc'
         namespace: default
       syncPolicy:
         automated:
           prune: true
           selfHeal: true
     ```
   - Apply the application configuration:
     ```bash
     kubectl apply -f argo-cd-application.yaml
     ```

#### 6. Sync and Deploy
   - In the Argo CD UI, select your application and click the "Sync" button to deploy it.
   - Alternatively, sync using the CLI:
     ```bash
     argocd app sync tic-tac-toe
     ```

#### 7. Verify the Deployment
   - Check the status of Pods and Services:
     ```bash
     kubectl get pods
     kubectl get services
     ```
   - Forward a service port to access the application locally:
     ```bash
     kubectl port-forward svc/<your-service-name> <local-port>:<service-port>
     ```

---

## Security Scanning

This project integrates **Trivy** to scan for vulnerabilities in the Docker image. The scan results are displayed in the CI/CD pipeline logs.

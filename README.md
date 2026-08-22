# ShopNow

ShopNow is a full-stack e-commerce capstone project. Customers can browse products, use a cart, place an order, and receive a collection token. Administrators can find orders, update their status, confirm collection, and view sales information.

## Project at a glance

| Component | Technology | Purpose |
|---|---|---|
| Customer app | React 18, Nginx | Catalog, cart, checkout, order confirmation |
| Admin app | React 18, Nginx | Order management and dashboard |
| Backend | Node.js, Express, Mongoose | REST API and business logic |
| Database | MongoDB 7 | Products, users, and invoices |
| Delivery | Docker, Jenkins, ECR, EKS | Build and deploy the application |

## Architecture

```text
Customer  -> Customer React application --\
                                            -> Express REST API -> MongoDB
Admin     -> Admin React application -------/
```

### How it works

1. The customer or administrator opens the application.
2. React displays the interface and sends requests to `/api`.
3. Express validates each request and runs the business logic.
4. Mongoose reads or updates MongoDB.
5. The backend returns JSON and React updates the page.

### Order flow

```text
Customer selects products
  -> Customer app sends checkout details
  -> Backend creates the invoice
  -> MongoDB saves the order and updates stock
  -> Customer receives an order and collection token
  -> Administrator finds the order
  -> Administrator updates its status
```

## Main features

- Browse and filter the product catalog
- Add products to a shopping cart
- Submit checkout details and create an invoice
- Generate an order collection token
- View customer orders by email
- Search and manage orders from the admin application
- Update order and payment status
- View product and order analytics

## Project structure

```text
shopNow/
|-- frontend/          # Customer React app
|-- admin/             # Administrator React app
|-- backend/           # Express REST API
|-- docker-compose.yml # Local environment
|-- Jenkinsfile        # CI/CD pipeline
|-- deploy-aws-eks.*   # EKS deployment scripts
`-- README.md
```

## Run with Docker

### 1. Clone the project

```bash
git clone https://github.com/harishmsgit/shopNow.git
cd shopNow
```

### 2. Start all services

```bash
docker compose up --build -d
docker compose ps
```

### 3. Open the applications

| Service | URL |
|---|---|
| Customer app | <http://localhost:3000> |
| Admin app | <http://localhost:3002> |
| Backend health | <http://localhost:5000/api/health> |

### 4. Test the API

```bash
curl http://localhost:5000/api/health
curl http://localhost:5000/api/products
curl http://localhost:5000/api/categories
curl http://localhost:5000/api/analytics/dashboard
```

### 5. View logs or stop

```bash
docker compose logs -f backend
docker compose down
```

## Run without Docker

Start MongoDB locally, then use a separate terminal for each component:

```bash
# Backend
cd backend
npm ci
cp .env.example .env
npm start
```

```bash
# Customer app
cd frontend
npm ci
npm start
```

```bash
# Admin app
cd admin
npm ci
npm start
```

On PowerShell, use `Copy-Item .env.example .env`. Set a valid `MONGODB_URI` in the backend `.env` file.

## Main API endpoints

Base URL: `http://localhost:5000/api`

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Backend health |
| `GET/POST` | `/products` | List or create products |
| `PUT/DELETE` | `/products/:id` | Update or delete a product |
| `POST` | `/invoices` | Create an order |
| `GET` | `/invoices` | List or search orders |
| `GET` | `/invoices/token/:token` | Find an order by token |
| `PUT` | `/invoices/:id/status` | Update order status |
| `PUT` | `/invoices/token/:token/collect` | Confirm collection |
| `GET` | `/users/:email/orders` | Customer order history |
| `GET` | `/analytics/dashboard` | Dashboard totals |
| `GET` | `/categories` | Product categories |

## Deployment flow

```text
Code commit
  -> Jenkins builds ShopNow
  -> Docker images are created
  -> Images are pushed to Amazon ECR
  -> Images are deployed to AWS EKS
  -> Kubernetes checks application health
```

The companion [HeroVired Infrastructure](https://github.com/harishmsgit/herovired-infra) project provisions EKS and deploys the three application images.

## How ShopNow, AWS, EKS, Nginx, and infrastructure work together

```text
shopNow code + herovired-infra configuration
  -> Jenkins pipeline
  -> Amazon ECR stores the ShopNow images
  -> Terraform creates AWS EKS
  -> Kubernetes runs the ShopNow containers
  -> AWS Load Balancer receives user traffic
  -> Nginx Ingress selects frontend, admin, or backend
  -> Backend connects privately to MongoDB
```

### Deployment flow

1. The `shopNow` repository contains the customer UI, admin UI, backend, and Dockerfiles.
2. The `herovired-infra` repository contains Terraform, Ansible, Kubernetes manifests, and the infrastructure pipeline.
3. Jenkins builds three ShopNow images: `frontend`, `admin`, and `backend`.
4. Jenkins pushes those images to Amazon ECR.
5. Terraform creates the AWS network, IAM permissions, and Amazon EKS cluster.
6. Kubernetes pulls the ShopNow images from ECR and starts the pods in `shopnow-ns`.
7. Kubernetes Services give the pods stable internal addresses.
8. The Nginx Ingress Controller receives traffic from the AWS load balancer and sends it to the correct service.

### Live request flow

| Request | Nginx ingress destination | Result |
|---|---|---|
| Customer application path | `frontend-service:80` | Container Nginx serves the compiled customer React app |
| Admin application path | `admin-service:80` | Container Nginx serves the compiled admin React app |
| API path | `backend-service:5000` | Express processes the API request |

For an API request, the full path is:

```text
Browser
  -> AWS Load Balancer
  -> Nginx Ingress Controller
  -> backend-service:5000
  -> Express backend pod
  -> MongoDB service:27017
  -> JSON response back to the browser
```

The frontend and admin containers also use Nginx to serve the compiled React files and provide single-page-application routing. MongoDB is an internal service and is not exposed through the public load balancer.

## Screenshots

### Project architecture

![ShopNow architecture and infrastructure overview](screenshots/architecture.png)

### Customer application

![ShopNow customer product catalog and checkout](screenshots/customer-app.png)

### Admin application

![ShopNow admin order-management screen](screenshots/admin-app.png)

The screenshots above were cropped to exclude personal and secret demo values from the original POC evidence.

## Notes consolidated from `docs/`

This README now contains the useful project information that was previously split across the documentation folder:

### Application contract

- The application repository owns the React and Express source, Dockerfiles, dependency lockfiles, and API behavior.
- The infrastructure repository owns AWS, EKS, ingress, Kubernetes manifests, secrets integration, and monitoring.
- The frontend and admin use `REACT_APP_API_BASE_URL` to reach the `/api` endpoint.
- The backend listens on port `5000` and uses `MONGODB_URI` for durable data.
- A release produces separate immutable frontend, admin, and backend images.

### Data and order rules

- Products contain catalog, price, category, rating, image, and stock information.
- Invoices contain customer, item, total, payment, and fulfillment information.
- Invoice numbers and collection tokens are unique.
- Valid order states are `pending`, `ready`, `collected`, and `cancelled`.
- Collecting an order changes its status to `collected` and payment to `paid`.

### Deployment approach

1. Jenkins installs dependencies from the committed lockfiles.
2. React production builds and the backend image are created.
3. The three images are tagged with a release or Git commit.
4. Images are pushed to Amazon ECR.
5. The infrastructure pipeline deploys them to EKS.
6. Backend health, application routes, and Kubernetes rollout status are checked.

### Quick troubleshooting

| Problem | Check |
|---|---|
| UI opens but data does not load | Backend logs, port `5000`, and `REACT_APP_API_BASE_URL` |
| Backend cannot connect to MongoDB | `MONGODB_URI`; use `mongo` inside Docker and `localhost` from the host |
| Product catalog is empty | Call `/api/products` and confirm MongoDB contains product data |
| React page returns 404 after refresh | Nginx SPA fallback and the configured application base path |
| EKS application is unavailable | Pods, Services, ingress, rollout status, and namespace events |

## References

- [React](https://react.dev/)
- [Express](https://expressjs.com/)
- [MongoDB](https://www.mongodb.com/docs/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Kubernetes](https://kubernetes.io/docs/)

## License

See [LICENSE](LICENSE).

# AWS Mono Web Deployment Lab

> Stack: ASP.NET Core Web API, React + TypeScript, MySQL.
> Goal: deploy a simple mono web app on AWS before moving to microservices.
> Cost rule: prefer free/cheap services, stop/delete resources after practice.

---

## Phase 0 - Cost Safety

- [ ] Pick one AWS Region for the whole lab.
- [ ] Open AWS Billing dashboard.
- [ ] Check Free Tier / credit eligibility.
- [ ] Create AWS Budget.
- [ ] Add billing alert email.
- [ ] Decide max lab spend, for example USD 5-20.
- [ ] Review cleanup checklist before creating resources.

---

## Phase 1 - Prepare Application

Backend:

- [ ] Confirm ASP.NET API runs locally.
- [ ] Confirm app uses MySQL provider.
- [ ] Move deploy-specific config to environment variables.
- [ ] Keep secrets out of Git.
- [ ] Add health endpoint, for example `/health`.
- [ ] Configure CORS for local and deployed frontend origins.
- [ ] Confirm EF Core migrations work.

Frontend:

- [ ] Confirm React + TypeScript app runs locally.
- [ ] Configure API base URL through env variable.
- [ ] Confirm production build works.
- [ ] Confirm built frontend can call backend API.

Database:

- [ ] Prepare MySQL database name/user/password.
- [ ] Prepare migration command.
- [ ] Prepare simple backup/export command.

---

## Phase 2 - Dockerize Application

- [ ] Create backend `Dockerfile`.
- [ ] Create frontend production build process.
- [ ] Choose reverse proxy: Caddy or Nginx.
- [ ] Create `docker-compose.yml`.
- [ ] Add services:
  - `reverse-proxy`
  - `api`
  - `mysql`
- [ ] Add persistent MySQL volume.
- [ ] Add `.env` for runtime config.
- [ ] Test `docker compose up -d` locally if possible.
- [ ] Test frontend.
- [ ] Test API health endpoint.
- [ ] Restart containers and verify MySQL data persists.

Target local routing:

```text
http://localhost        -> frontend
http://localhost/api    -> API, or use separate API host/port
mysql:3306             -> internal only
```

---

## Phase 3 - AWS Networking

- [ ] Create VPC, for example `10.0.0.0/16`.
- [ ] Create public subnet, for example `10.0.1.0/24`.
- [ ] Create Internet Gateway.
- [ ] Attach Internet Gateway to VPC.
- [ ] Create public route table.
- [ ] Add route `0.0.0.0/0 -> Internet Gateway`.
- [ ] Associate route table with public subnet.
- [ ] Create EC2 Security Group.

Inbound rules:

```text
22   -> your IP only
80   -> 0.0.0.0/0
443  -> 0.0.0.0/0
```

Do not expose:

```text
3306
5000
8080
```

---

## Phase 4 - EC2 Setup

- [ ] Launch EC2 in public subnet.
- [ ] Use free-tier eligible/small instance if possible.
- [ ] Use Amazon Linux 2023 or Ubuntu LTS.
- [ ] Configure 20-30 GB gp3 EBS.
- [ ] Attach Security Group.
- [ ] SSH into EC2.
- [ ] Install Docker.
- [ ] Install Docker Compose plugin.
- [ ] Enable Docker on boot.
- [ ] Create app directory, for example `/opt/ielts-app`.
- [ ] Verify:

```bash
free -h
df -h
docker --version
docker compose version
```

---

## Phase 5 - Manual Deploy On EC2

- [ ] Copy or clone project to EC2.
- [ ] Copy `.env` to EC2.
- [ ] Copy reverse proxy config.
- [ ] Build images on EC2 or pull from registry.
- [ ] Run:

```bash
docker compose up -d
docker compose ps
docker compose logs api
```

- [ ] Run database migrations.
- [ ] Test API health endpoint.
- [ ] Test frontend by EC2 public IP.
- [ ] Restart EC2.
- [ ] Verify containers recover.
- [ ] Verify MySQL data persists.

---

## Phase 6 - Domain And HTTPS

- [ ] Point domain/subdomain to EC2 public IP.
- [ ] Configure Caddy or Nginx.
- [ ] Serve frontend through domain.
- [ ] Route API traffic to ASP.NET container.
- [ ] Enable HTTPS.
- [ ] Verify HTTP redirects to HTTPS.
- [ ] Verify raw container ports are not public.

Example routing:

```text
https://example.com      -> React frontend
https://api.example.com  -> ASP.NET API
```

---

## Phase 7 - MySQL Strategy

Mode A - MySQL container on EC2:

- [ ] Use MySQL Docker image.
- [ ] Store data in Docker volume or host-mounted path.
- [ ] Keep port 3306 internal.
- [ ] Run migrations.
- [ ] Test CRUD from app.
- [ ] Export backup with `mysqldump`.

Mode B - Optional RDS MySQL experiment:

- [ ] Verify free-tier/credit eligibility.
- [ ] Create DB subnet group.
- [ ] Create small Single-AZ RDS MySQL.
- [ ] Disable public access.
- [ ] Allow 3306 only from EC2 Security Group.
- [ ] Update backend connection string.
- [ ] Run migrations.
- [ ] Test app.
- [ ] Delete RDS after experiment.
- [ ] Delete final snapshot if not needed.

---

## Phase 8 - Optional S3

Use only if needed for frontend hosting practice or app file uploads.

Frontend static hosting:

- [ ] Build frontend.
- [ ] Upload build output to S3.
- [ ] Optionally put CloudFront in front.
- [ ] Configure custom domain/HTTPS if practicing this path.

Application uploads:

- [ ] Create S3 bucket.
- [ ] Keep public access blocked unless intentionally needed.
- [ ] Attach IAM role to EC2.
- [ ] Grant minimum S3 permissions.
- [ ] Configure app to use IAM role.
- [ ] Test upload/read/delete.
- [ ] Add lifecycle rule if useful.

---

## Phase 9 - ECR

- [ ] Create ECR repository for API image.
- [ ] Optionally create ECR repository for frontend image.
- [ ] Authenticate Docker to ECR.
- [ ] Build image.
- [ ] Tag image with Git commit SHA.
- [ ] Push image to ECR.
- [ ] Pull image from EC2.
- [ ] Update Compose to use ECR image.
- [ ] Add ECR lifecycle policy.

---

## Phase 10 - CloudWatch

- [ ] Create log group for API.
- [ ] Send app/container logs to CloudWatch if practicing log integration.
- [ ] Set log retention, for example 7 or 14 days.
- [ ] Monitor EC2 CPU/network/status checks.
- [ ] Optionally install CloudWatch Agent for memory/disk metrics.
- [ ] Create simple dashboard.
- [ ] Create basic alarm for EC2 status check failure.

---

## Phase 11 - CI

- [ ] Create `.github/workflows/ci.yml`.
- [ ] Trigger on pull request.
- [ ] Trigger on push to main/develop.
- [ ] Run backend restore/build/test.
- [ ] Run frontend install/build.
- [ ] Build Docker images.
- [ ] Fail pipeline when build/test fails.

---

## Phase 12 - GitHub Actions To AWS With OIDC

- [ ] Create GitHub OIDC provider in AWS IAM.
- [ ] Create deployment IAM role.
- [ ] Restrict trust policy to your repository.
- [ ] Restrict branch/environment if possible.
- [ ] Grant required ECR permissions.
- [ ] Avoid permanent AWS access keys in GitHub Secrets.
- [ ] Test GitHub Actions can assume AWS role.

---

## Phase 13 - CD To EC2

- [ ] Build image in GitHub Actions.
- [ ] Push image to ECR with commit SHA tag.
- [ ] Deploy to EC2 manually first:

```bash
docker compose pull
docker compose up -d
```

- [ ] Add automatic EC2 deploy later using SSH or AWS Systems Manager.
- [ ] Run health check after deployment.
- [ ] Keep previous image tag for rollback.
- [ ] Test rollback.

---

## Phase 14 - Optional ECS With EC2 Capacity

Do this only after EC2 + Docker Compose is clear.

- [ ] Create ECS cluster.
- [ ] Register EC2 as ECS capacity.
- [ ] Create API task definition.
- [ ] Configure environment variables.
- [ ] Configure CloudWatch logs.
- [ ] Create ECS service with desired count 1.
- [ ] Deploy image from ECR.
- [ ] Kill task and verify ECS replaces it.
- [ ] Deploy new task definition revision.
- [ ] Test rollback to previous revision.

---

## Daily Lab Workflow

Start:

- [ ] Start EC2.
- [ ] Wait for status checks.
- [ ] Check containers.
- [ ] Open frontend/API.

Finish:

- [ ] Export DB backup if needed.
- [ ] Stop EC2.
- [ ] Confirm EC2 state is `stopped`.
- [ ] Check Billing/Cost Explorer.

---

## Cleanup Checklist

EC2:

- [ ] Stop or terminate EC2.
- [ ] Delete unattached EBS volumes.
- [ ] Delete snapshots not needed.
- [ ] Release Elastic IP if created.

RDS:

- [ ] Delete RDS instance if used.
- [ ] Delete final snapshot if not needed.
- [ ] Delete DB subnet group if unused.

Networking:

- [ ] Delete Security Groups.
- [ ] Delete route table/subnet/VPC if no longer needed.
- [ ] Delete Route 53 hosted zone if created and unused.

Storage/observability:

- [ ] Empty/delete S3 bucket if used.
- [ ] Delete unused ECR images/repositories.
- [ ] Delete CloudWatch log groups or reduce retention.
- [ ] Review Cost Explorer.

---

## Definition Of Done

- [ ] Frontend works through HTTPS.
- [ ] API works through HTTPS.
- [ ] API connects to MySQL.
- [ ] Data persists after restart.
- [ ] Only 22/80/443 are public.
- [ ] SSH is restricted to your IP.
- [ ] Images can be built reproducibly.
- [ ] Optional: images are pushed to ECR.
- [ ] Optional: CI builds successfully.
- [ ] Optional: CD deploys new version.
- [ ] Rollback path is known.
- [ ] Cleanup path is tested.

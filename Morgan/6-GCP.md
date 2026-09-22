# what GCP is used for in this project

**Identity / Auth (GCIP / Firebase)**

The repo can create a Google Cloud Identity Platform tenant and config via module.gcip / modules/id_platform_config. This provisions GCIP and wiring so the app can use Firebase/GCIP for user authentication.

A Cloud Function (beforeCreate) is deployed to run on user creation (blocking function) to enforce custom logic (see modules/id_platform_config/cloud_function).
GCP project & CI integration

modules/gcp_project provisions Google Projects, enables required APIs, creates a service account and a Workload Identity Pool/Provider so GitLab CI can authenticate to GCP (OIDC) securely for CI/CD tasks.

**Secrets & runtime usage**

Firebase/GCIP credentials (API key, admin private key) are exposed into app environment/secrets (see module.secrets entries like FIREBASE_QA_API_KEY and FIREBASE_ADMIN_PRIVATE_KEY).
workload_cluster annotations include gcipProjectId/gcipTenant so ArgoCD-deployed apps know which GCIP instance to use.
Why it’s here (purpose)

Provide user authentication (sign-in, user lifecycle hooks) backed by Google’s identity platform and Cloud Functions for custom logic while keeping main application and data plane on AWS.
Allow GitLab CI to perform GCP operations securely via Workload Identity.


## Diagram

GCIP Flow (quick diagram)

Browser → Web app (served from EKS via ALB/NLB)
Web app → GCIP (Identity Platform) — sign-in / sign-up / token flows
On user create: GCIP → Cloud Function (beforeCreate HTTP trigger, public)
Cloud Function → GCIP (returns allow/deny or modifies user data)
GCIP → Web app (user created / token issued)
ASCII diagram:

Browser
↓
Web App (EKS behind ALB/NLB)
↓ (auth request)
GCIP / Firebase Identity Platform
↓ (blocking trigger)
Cloud Function (google_cloudfunctions_function.before_create — public)
↓ (response)
GCIP
↓ (token / user)
Web App

Key points

The Cloud Function is deployed in modules/id_platform_config and is exposed publicly (google_cloudfunctions_function_iam_member member = allUsers) so GCIP can call it as a blocking function.
workload_cluster annotations include gcipProjectId and gcipTenant so apps deployed via ArgoCD know which GCIP tenant to use.
Secrets/API keys for GCIP (FIREBASE_QA_API_KEY, FIREBASE_ADMIN_PRIVATE_KEY) are stored in module.secrets and injected into apps as needed.
GPT-5 mini • 1x
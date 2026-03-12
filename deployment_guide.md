# Deployment & Integration Guide (Person 5)

## 1. GitHub Actions (Automated Testing)
The `.github/workflows/ci.yml` file is already created. To activate it:
1. Stage all the files we created: `git add .`
2. Commit your changes: `git commit -m "Add CI, render blueprint, and env configs"`
3. Push to your repository: `git push origin main`
* **What happens next:** Go to your repository on GitHub $\rightarrow$ **"Actions"** tab. You will see a workflow running automatically that installs dependencies, checks Python syntax, and typechecks the Node.js backend. It runs on every future push.

## 2. Deploying the Backend on Render
The `render.yaml` file acts as an infrastructure-as-code blueprint, meaning Render automates the deployment settings using this file.
1. Go to your [Render Dashboard](https://dashboard.render.com).
2. Click **"New +"** and select **"Blueprint"** (Not "Web Service").
3. Connect your GitHub repository containing the Code.
4. Render will read the `render.yaml` file automatically to provision the service named *career-path-backend*.
5. On the Render Dashboard, go to your new service $\rightarrow$ **Environment**, and manually populate the "Sync: False" environment variables from `.env`. (e.g. `DATABASE_URL`, `JWT_SECRET`, `REDIS_URL`, `FRONTEND_URL`). Everything else is pre-configured.

## 3. Testing with Postman
I have created `postman_collection.json`. Here is how you use it to test everything:
1. Open the **Postman** app.
2. Click **"Import"** (located near "New" at the top left).
3. Select the `postman_collection.json` file from your Desktop.
4. Open the newly imported folder "Career Path Generator".
5. In Postman, go to **Variables** on the left or top right for this collection.
6. Make sure `api_url` is set to `http://localhost:4000` (or your Render URL once deployed).
7. Run the requests in order (1 $\rightarrow$ 5).
8. *Note*: After you run **"2. Auth - Login"**, you need to copy the `token` from the response and paste it into the `jwt_token` variable in Postman to unlock the profile and roadmap endpoints.

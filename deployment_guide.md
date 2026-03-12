# Deployment & Integration Guide (Person 5)

## 1. GitHub Actions (Automated Testing)
The `.github/workflows/ci.yml` file is already created. To activate it:
1. Stage all the files we created: `git add .`
2. Commit your changes: `git commit -m "Add CI, render blueprint, and env configs"`
3. Push to your repository: `git push origin main`
* **What happens next:** Go to your repository on GitHub $\rightarrow$ **"Actions"** tab. You will see a workflow running automatically that installs dependencies, checks Python syntax, and typechecks the Node.js backend. It runs on every future push.

## 2. Deploying the RAG Service to Hugging Face Spaces
Person 3 already built the **RAG Service** with a `Dockerfile` exposed on port `8000`. Here's exactly how to deploy it on Hugging Face (HF) Spaces:
1. Go to [Hugging Face Spaces](https://huggingface.co/spaces) and click **"Create new Space"**.
2. **Space name**: e.g., `career-path-generator-rag`.
3. **Select the Space SDK**: Choose **"Docker"** (then select "Blank").
4. **Hardware**: "CPU Basic" (Free) is fine for your current ChromaDB + Groq setup.
5. Under **"Space settings"** $\rightarrow$ **"Variables and secrets"**, add the following Secrets (from your `.env`):
   - `GROQ_API_KEY` = `[your key]`
   - *(No need to set PORT or HOST, HF Docker Spaces figure this out automatically, but HF maps internal port `7860` by default. Change the end of your Dockerfile to `CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "7860"]` right before deploying, or set the `PORT` env var in HF settings to 8000).*
6. Clone the HF Space repository.
7. Copy the following files and folders from your root directory into the HF Space repository:
   - `main.py`, `models.py`, `config.py`, `requirements.txt`, `Dockerfile`
   - **`rag/`** (Crucial: contains the RAG engine logic)
   - `prompts/`
   - `chroma_data/` (Optional but highly recommended so it doesn't re-embed from scratch)
8. `git add .`, `git commit -m "Deploy RAG"`, and `git push`.
9. The Space will build the Docker container and start your FastAPI server. The public URL will be `https://[your-username]-career-path-generator-rag.hf.space`.

## 3. Deploying the Backend on Vercel (Free)
We configured Vercel Serverless Functions for the backend, meaning you do not need to use Render or provide a credit card.
1. Go to your [Vercel Dashboard](https://vercel.com/dashboard).
2. Click **"Add New..."** $\rightarrow$ **"Project"** and connect your GitHub repository.
3. **CRITICAL STEP**: Under **"Root Directory"**, click **"Edit"** and select the `/backend` folder.
4. Under **"Environment Variables"**, manually populate the environment variables from your `.env`:
   - `DATABASE_URL`
   - `JWT_SECRET`
   - `REDIS_URL`
   - `RAG_SERVICE_URL` (Make sure this points to the Hugging Face URL you generated in Step 2!)
5. Click **Deploy**. Vercel will build the Prisma client and deploy your Express backend for free.
6. Once deployed, note down the URL Vercel gives you (e.g., `https://career-path-backend.vercel.app`).

## 4. Testing with Postman
I have created `postman_collection.json`. Here is how you use it to test everything:
1. Open the **Postman** app.
2. Click **"Import"** (located near "New" at the top left).
3. Select the `postman_collection.json` file from your Desktop.
4. Open the newly imported folder "Career Path Generator".
5. In Postman, go to **Variables** on the left or top right for this collection.
6. Make sure `api_url` is set to `http://localhost:4000` (or your new Vercel backend URL).
7. Run the requests in order (1 $\rightarrow$ 5).
8. *Note*: After you run **"2. Auth - Login"**, you need to copy the `token` from the response and paste it into the `jwt_token` variable in Postman to unlock the profile and roadmap endpoints.

## 5. Deploying the Frontend on Vercel
Now that Person 1 has pushed the `career-path-gen` Next.js frontend, you can deploy it:
1. Go to your Vercel Dashboard and click **"Add New..."** $\rightarrow$ **"Project"**.
2. Connect your GitHub repository.
3. Under **"Root Directory"**, click **"Edit"** and select the `/career-path-gen` folder.
4. Under **"Environment Variables"**, add:
   - `NEXT_PUBLIC_API_URL` = `[your Vercel BACKEND URL from Step 3]`
5. Click **Deploy**. Vercel will automatically run `npm run build` inside that directory.
6. *Important*: Once the frontend finishes deploying, go back to your Backend Vercel project $\rightarrow$ Settings $\rightarrow$ Environment Variables $\rightarrow$ Add `FRONTEND_URL` and set it to your Frontend's live URL. This ensures CORS allows communication!

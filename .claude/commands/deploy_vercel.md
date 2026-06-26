Deploy this project to Vercel production.

Steps:
1. Run `npm run build` to build the project (output goes to `./build`)
2. Run `vercel --prod --yes` to deploy to production
3. Report the production URL when done

If the Vercel CLI is not installed, run `npm install -g vercel` first.
If not logged in, run `vercel whoami` which will trigger the login flow.
If the project is not linked, run `vercel link --yes` first.

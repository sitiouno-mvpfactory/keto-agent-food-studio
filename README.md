# Keto Agent Food

Transform PDFs, images, and EPUBs into clean, normalized Markdown for AI ingestion. Built for developers and AI agents, it features a robust REST API, MCP Server integration, secure Stripe billing, and real-time webhooks.

## Architecture Overview

```text
[Client / AI Agent] ---> (HTTPS / REST) ---> [Cloud Run: Keto Agent Food]
                                                    |
                                     +--------------+--------------+
                                     |                             |
                              [Cloud SQL DB]               [Stripe Billing]
                               (quien-prod)              (Usage-based/Metered)
```

## Environment Variables Reference

To run this service locally or in production, configure the following environment variables:

* `DOMAIN`: `keto-agent-food.x53.ai`
* `PROJECT_NAME`: `Keto Agent Food`
* `DB_USER`: `keto_agent_food_user`
* `DB_PASS`: *(Retrieve from GCP Secret Manager `keto_agent_food-db-password`)*
* `DB_NAME`: `keto_agent_food`
* `DB_CONNECTION`: `test-agents-ai-app:us-central1:quien-prod`
* `STRIPE_API_KEY`: Your Stripe secret key
* `STRIPE_WEBHOOK_SECRET`: Webhook signing secret from Stripe

## Deployment Instructions

This project is configured to be deployed to Google Cloud Run. 

* **Service Name:** `keto-agent-food-studio`
* **Live URL:** https://keto-agent-food-studio-test-agents-ai-app.us-central1.run.app

Manual deployments can be triggered using the `gcloud` CLI:
```bash
gcloud run deploy keto-agent-food-studio --source . --region us-central1
```

## DNS Configuration Steps

To map the custom domain, you must update your DNS provider with the following records:

* **Type:** `CNAME`
* **Name/Host:** `keto-agent-food.x53.ai`
* **Target/Value:** `ghs.googlehosted.com`

*(Note: If configuring at the apex level, use an A record pointing to Google's load balancer IPs as directed in the Google Cloud Console).*

## Stripe Webhook Setup

Since this product relies on usage-based (metered) pricing per Megabyte processed, Stripe must be fully configured:

1. Go to your **Stripe Dashboard** -> **Developers** -> **Webhooks**.
2. Add a new endpoint: `https://keto-agent-food.x53.ai/webhooks/stripe`
3. Listen for the following events:
   - `checkout.session.completed`
   - `invoice.paid`
4. Copy the resulting **Webhook Secret** and update `src/config/stripe_config.py` (or set the `STRIPE_WEBHOOK_SECRET` environment variable).
5. Ensure you have created a Product named "Keto Agent Food" in Stripe with a recurring, metered pricing model.

## Monitoring and Troubleshooting

* **Logs:** Application logs are available in the Google Cloud Console under **Cloud Run** -> `keto-agent-food-studio` -> **Logs**.
* **Database Access:** Use the Cloud SQL Auth Proxy to securely connect to your database instance locally using the connection name `test-agents-ai-app:us-central1:quien-prod`.
* **Billing:** Monitor Stripe webhook deliveries in the Stripe developer dashboard to ensure successful usage reporting.

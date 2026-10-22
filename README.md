# gcp-devops-project

Deploys the Docker [example-voting-app](https://github.com/dockersamples/example-voting-app) on GKE.

| Component | Image                                   | Role                                            |
| --------- | --------------------------------------- | ----------------------------------------------- |
| vote      | `dockersamples/examplevotingapp_vote`   | Python/Flask frontend where users vote          |
| redis     | `redis:alpine`                          | Queue for incoming votes                        |
| worker    | `dockersamples/examplevotingapp_worker` | .NET worker moving votes from Redis to Postgres |
| db        | `postgres:15-alpine`                    | Persistent vote storage                         |
| result    | `dockersamples/examplevotingapp_result` | Node.js frontend showing live results           |

## Run locally

```sh
cp .env.example .env   # optional; compose falls back to postgres/postgres
docker compose up
```

- Vote: http://localhost:8080
- Result: http://localhost:8081

## Deploy to GKE

```sh
gcloud container clusters get-credentials <CLUSTER> --region <REGION> --project <PROJECT>
kubectl apply -f k8s/namespace.yaml

# credentials come from your shell environment, never from the manifests
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=postgres
kubectl -n voting-app create secret generic db-credentials \
  --from-literal=POSTGRES_USER="$POSTGRES_USER" \
  --from-literal=POSTGRES_PASSWORD="$POSTGRES_PASSWORD"

kubectl apply -f k8s/
kubectl -n voting-app get pods
kubectl -n voting-app get svc vote result   # wait for EXTERNAL-IP
```

`vote` and `result` are exposed through GCP external load balancers; `redis` and `db` are cluster-internal.

## Cleanup

```sh
kubectl delete namespace voting-app
```

## Notes

- No credentials are stored in the manifests: `k8s/db.yaml` reads the `db-credentials` Secret, which you create
  from environment variables (see above). `.env` is gitignored.
- The prebuilt `worker` and `result` images have `postgres`/`postgres` hardcoded. Using any other value will
  break them unless you rebuild those images from the upstream source and pass the credentials in.
- The previous Flask hello-world app is kept in `legacy-flask-app/` and can be deleted.

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
docker compose up
```

- Vote: http://localhost:8080
- Result: http://localhost:8081

## Deploy to GKE

```sh
gcloud container clusters get-credentials <CLUSTER> --region <REGION> --project <PROJECT>
kubectl apply -f k8s/namespace.yaml
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

- The Postgres credentials in `k8s/db.yaml` are the sample defaults (`postgres`/`postgres`), which the prebuilt
  worker and result images expect. Fine for a demo, not for production.
- The previous Flask hello-world app is kept in `legacy-flask-app/` and can be deleted.

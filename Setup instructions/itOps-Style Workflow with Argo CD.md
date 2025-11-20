GitOps-Style Workflow with Argo CD (Optional)

Install Argo CD

      kubectl create namespace argocd
      kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
      kubectl port-forward svc/argocd-server -n argocd 8080:443
Connect GitHub Repo

Push your Kubernetes manifests to a GitHub repo.

Create an Argo CD application pointing to this repo.

Argo CD will sync automatically on changes.

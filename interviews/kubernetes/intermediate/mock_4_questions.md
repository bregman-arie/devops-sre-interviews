# Kubernetes - Intermediate Interview Mock #4

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of Kubernetes CI/CD integration, GitOps, application lifecycle management, and deployment strategies.

---

## 🧠 Section 1: Core Questions

1. How do you implement **GitOps** workflows with Kubernetes? What are the benefits and challenges? [📖 Answer](mock_4_answers.md#1-how-do-you-implement-gitops-workflows-with-kubernetes-what-are-the-benefits-and-challenges)
2. Explain **Helm Charts** and how they simplify Kubernetes application management. [📖 Answer](mock_4_answers.md#2-explain-helm-charts-and-how-they-simplify-kubernetes-application-management)
3. What are **Kustomize** overlays and how do they help manage configurations across environments? [📖 Answer](mock_4_answers.md#3-what-are-kustomize-overlays-and-how-do-they-help-manage-configurations-across-environments)
4. How do you implement **Progressive Delivery** (Canary, Blue-Green) using Kubernetes-native tools? [📖 Answer](mock_4_answers.md#4-how-do-you-implement-progressive-delivery-canary-blue-green-using-kubernetes-native-tools)
5. What are **Operators** and how do they automate application lifecycle management? [📖 Answer](mock_4_answers.md#5-what-are-operators-and-how-do-they-automate-application-lifecycle-management)
6. How do you manage **secrets and configurations** securely in CI/CD pipelines? [📖 Answer](mock_4_answers.md#6-how-do-you-manage-secrets-and-configurations-securely-in-cicd-pipelines)
7. Explain **Multi-environment deployment strategies** (dev/staging/prod) in Kubernetes. [📖 Answer](mock_4_answers.md#7-explain-multi-environment-deployment-strategies-devstaging-prod-in-kubernetes)
8. How do you implement **Feature Flags** and **A/B Testing** in Kubernetes deployments? [📖 Answer](mock_4_answers.md#8-how-do-you-implement-feature-flags-and-ab-testing-in-kubernetes-deployments)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your development team is transitioning from traditional deployment methods to a cloud-native approach. They need to implement a complete CI/CD pipeline that supports:

- Multiple microservices with different release cycles
- Automated testing and security scanning
- Progressive deployment strategies to minimize risk
- Environment-specific configurations (dev/staging/prod)  
- Rollback capabilities and deployment approval workflows
- Integration with existing tools (Jenkins, GitLab, monitoring)

The team wants to adopt GitOps principles while ensuring developer productivity and operational safety. Design a comprehensive CI/CD strategy that includes tooling recommendations, workflow design, and governance policies.

[📖 Answer](mock_4_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A development team is struggling with their current deployment process. They have multiple issues with their Helm-based deployment pipeline that's causing frequent production incidents and developer frustration.

**Current Setup Issues:**

```yaml
# Problematic Helm values structure
# values-prod.yaml
global:
  image:
    tag: latest  # Problem: Using latest tag
  
app:
  replicas: 1    # Problem: No redundancy
  resources: {}  # Problem: No resource management
  
database:
  password: "hardcoded123"  # Problem: Hardcoded secrets
  
ingress:
  enabled: true
  host: "myapp.com"  # Problem: Same host for all environments

# Problematic CI pipeline (simplified)
stages:
  - build
  - deploy

build:
  script:
    - docker build -t myapp:latest .
    - docker push myapp:latest

deploy:  
  script:
    - helm upgrade --install myapp ./chart 
      --values values-prod.yaml
      --wait --timeout 60s
```

**Reported Problems:**
1. Deployments frequently fail due to resource constraints
2. No way to safely test changes before production
3. Rollbacks are manual and error-prone  
4. Database passwords are visible in Git repository
5. Same configuration is used across all environments
6. No automated testing in the pipeline
7. Developers can't track deployment status easily
8. Production incidents due to untested changes

Design an improved CI/CD workflow using modern GitOps practices, proper Helm configuration, and deployment safety measures.

[📖 Answer](mock_4_answers.md#-section-3-problem-solving---answer)

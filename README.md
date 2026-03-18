# 🚀 DevSecOps Supply Chain Pipeline with Kubernetes Enforcement
# 📌 Overview

This project demonstrates a complete end-to-end DevSecOps pipeline that secures containerized applications from build to deployment. It integrates vulnerability scanning, image signing, and runtime policy enforcement to ensure that only trusted and secure images are deployed into Kubernetes.

The goal of this project is to implement secure software supply chain practices similar to real-world production environments.

# 🏗️ Architecture Workflow
Code → Build → Scan → Push → Sign → Verify → Deploy → Runtime Policy Check
Flow Explanation:

Application code is built into a Docker image using Jenkins

Image is scanned using Trivy for vulnerabilities

Pipeline blocks if HIGH/CRITICAL vulnerabilities are found

Secure image is pushed to private Harbor registry

Image is signed using Cosign (cryptographic signing)

Signature is verified in the pipeline before deployment

Kubernetes pulls the image using imagePullSecrets

Kyverno policy validates image signatures at runtime (Audit mode)

# 🔧 Tech Stack

CI/CD: Jenkins

Containerization: Docker

Container Registry: Harbor

Security Scanning: Trivy

Image Signing: Cosign

Orchestration: Kubernetes

Policy Enforcement: Kyverno

# 🔐 Security Layers Implemented
1. Vulnerability Scanning

Integrated Trivy into pipeline

Blocks pipeline on HIGH/CRITICAL vulnerabilities

2. Image Signing

Cosign used to sign container images

Ensures image integrity and authenticity

3. Signature Verification

Pipeline verifies images before deployment

Prevents tampered images from being deployed

4. Kubernetes Runtime Policy

Kyverno policy checks image signatures

Currently configured in Audit mode

Logs warnings for unverified images

# ⚠️ Important Note (TLS Limitation)

This project uses a self-signed certificate for the Harbor registry.

Kubernetes and Kyverno cannot fully verify TLS

Hence, runtime enforcement is set to Audit mode instead of Enforce

In production, this would be solved using:

Trusted CA certificates

Proper TLS configuration

# 📂 Project Structure
.
├── Jenkinsfile
├── deployment.yaml
├── cosign-policy.yaml
├── Dockerfile
├── app/
└── README.md
# ⚙️ Setup Instructions (High-Level)
1. Build & Push Image
docker build -t <harbor-ip>/project/image:tag .
docker push <harbor-ip>/project/image:tag
2. Scan Image
trivy image --severity HIGH,CRITICAL <image>
3. Sign Image
cosign sign --key cosign.key <image>
4. Verify Signature
cosign verify --key cosign.pub <image>
5. Deploy to Kubernetes
kubectl apply -f deployment.yaml
6. Apply Kyverno Policy
kubectl apply -f cosign-policy.yaml
# 🧠 Key Learnings

Implementing secure CI/CD pipelines

Understanding container image vulnerabilities

Working with cryptographic image signing

Handling real-world TLS issues in private registries

Applying Kubernetes security policies

Debugging image pull and certificate errors

# 🚧 Challenges Faced

TLS verification failures with self-signed certificates

Kubernetes image pull issues from private registry

Kyverno policy compatibility with different versions

Integrating Cosign with private insecure registry

# 🎯 Future Improvements

Enable full enforcement mode in Kyverno

Use trusted CA certificates for Harbor

Integrate admission controllers with stricter policies

Add SBOM generation and verification

Implement GitOps (ArgoCD) for deployment

📣 Conclusion

This project showcases a practical implementation of DevSecOps principles, focusing on securing the software supply chain through scanning, signing, and policy enforcement. It highlights real-world challenges and demonstrates how to handle them effectively in a production-like setup.

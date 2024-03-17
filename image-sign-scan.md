Create a Docker file for the ethereum image

```Dockerfile
# Use the same base image as your pod container
FROM skylenet/ethstats-server:pow-latest

# Optionally, add any additional configurations or dependencies required for your application
```

1. **Build the Docker Image**:
     ```bash
     docker build -t my-ethereum-app:latest .
     ```
     - Replace `my-ethereum-app` with your desired image name and `latest` with the desired tag (e.g., version number).

2. **Tag the Docker Image**:
   - Once the image is scanned and signed (if applicable), tag it to associate it with a specific registry or repository. For example:
     ```bash
     docker tag my-ethereum-app:latest ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
     ```
   - Replace `ethereum.gm-nig-ltd.tech` with your actual registry address and `my-ethereum-app` with your image name.

By incorporating scanning and signing before pushing the image to Harbor, you ensure that the image is thoroughly checked for vulnerabilities and its integrity is verified before deployment.
3. **Create a Project in Harbor**:
   - Before pushing an image, you need to create a corresponding project in the Harbor interface. If you haven't done this yet, follow the steps to create a project. Projects help organize your images within Harbor.
project => private => eth-project
4. **Log in to Harbor**:
   - Open your terminal or command prompt.
   - Use the following command to log in to your Harbor registry:
     ```bash
     docker login ethereum.gm-nig-ltd.tech
     ```
     - You'll be prompted to enter your Docker registry username and password. Once authenticated, you'll be able to interact with your Harbor registry.

(base) simplymenah@Efemenas-MacBook-Pro SecuritySprint01-Pod1 % docker login ethereum.gm-nig-ltd.tech
Username: admin
Password: Harbor12345
Login Succeeded

5. **Push the Docker Image to Harbor**:
   - Assuming you've already built a Docker image (let's call it `my-ethereum-app:latest`), you can push it to your Harbor registry using the following command:
     ```bash
     docker push ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
     ```
     - Replace `my-ethereum-app` with your actual image name and `latest` with the desired tag (e.g., version number).

That's it! Your Docker image is now stored in Harbor. 🚀🐳

6. **Sign the Docker Image (Optional)**:
   - If required, sign the Docker image to ensure its integrity and authenticity. This step involves using cryptographic signatures to verify that the image hasn't been tampered with and originates from a trusted source.
   - Use a container scanning tool like Trivy or Clair to scan the Docker image for vulnerabilities. This step helps identify any security issues that need to be addressed before deployment. For example:
     ```bash
     brew install cosign

     cosign generate-key-pair
     Enter password for private key:
     Enter password for private key again:
     Private key written to cosign.key
     Public key written to cosign.pub

     docker image ls --digests # to get the SHA of the image
     docker pull ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
     ```
     It looks like you've successfully generated a key pair using Cosign. Now, you're listing Docker images along with their digests.

The output shows various Kubernetes-related images along with their digests. Additionally, there's an image named `my-ethereum-app` with the tag `latest`, but it doesn't have a digest associated with it.

If you want to sign this image using Cosign before pushing it to your registry, you need to sign it with the private key you generated and then verify the signature with the corresponding public key.

Here's a high-level overview of the process:

a. **Sign the Docker Image**:
   ```bash
   cosign sign --key cosign.key ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app@sha256:2b918f9ef3763ca571faebea27607096864a196d32c71d4f8ceaf6f27c387d22
   ```

b. **Verify the Signature** (Optional):
   ```bash
   cosign verify --key cosign.pub ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app@sha256:2b918f9ef3763ca571faebea27607096864a196d32c71d4f8ceaf6f27c387d22
   ```

c. **Push the Docker Image to the Registry**:
   ```bash
    docker push ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
   ```

7. **Scan the Docker Image for Vulnerabilities**:
     ```bash
     brew install trivy
     trivy image --ignore-unfixed --format cosign-vuln --output vuln.json ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app@sha256:2b918f9ef3763ca571faebea27607096864a196d32c71d4f8ceaf6f27c387d22
     ```

8. **Create a cosign attestation**
This command generates a cosign attestation for a Docker image, providing information about vulnerabilities detected in the image, and associates it with a specific image version identified by its SHA-256 digest.
A cosign attestation is essential for verifying the integrity, security, and compliance of Docker images. It provides cryptographic proof of these properties, ensuring trust, compliance, and accountability throughout the software supply chain.
     ```bash
     cosign attest --key cosign.key --type vuln --predicate vuln.json ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app@sha256:2b918f9ef3763ca571faebea27607096864a196d32c71d4f8ceaf6f27c387d22
     ```

9. **Verify that the container/image has an attestation**
This command generates a cosign attestation for a Docker image, providing information about vulnerabilities detected in the image, and associates it with a specific image version identified by its SHA-256 digest.
A cosign attestation is essential for verifying the integrity, security, and compliance of Docker images. It provides cryptographic proof of these properties, ensuring trust, compliance, and accountability throughout the software supply chain.
     ```bash
     cosign verify-attestation --key cosign.pub --type vuln ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app@sha256:2b918f9ef3763ca571faebea27607096864a196d32c71d4f8ceaf6f27c387d22
     ```

10. **Install a Policy agent**
By installing Kyverno policies, you can define rules that automatically validate and mutate Kubernetes resources based on your desired policies.
Kyverno helps enhance security, compliance, and best practices within your Kubernetes environment.
By enforcing policies, you can prevent misconfigurations, ensure consistent resource behavior, and maintain a secure cluster.
The provided commands set up Kyverno and its policies, allowing you to manage and enforce policies effectively.
     ```bash
    helm repo add kyverno https://kyverno.github.io/kyverno/
    helm repo update
    helm install kyverno-policies kyverno/kyverno-policies -n kyverno
    cat cosign.pub

    cosign verify --key cosign.pub ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app@sha256:2b918f9ef3763ca571faebea27607096864a196d32c71d4f8ceaf6f27c387d22
     ```
Create a policy.yaml file


8. **Add Descriptions to Repositories (Optional)**:
   - After pushing an image, the project administrator can add information to describe the repository.
   - In the Harbor interface, go into the repository, select the **Info** tab, and click the **Edit** button.
   - Enter a description and click **Save** to save the description.

9. **Download the Harbor Certificate (Optional)**:
   - Users can click the **Registry Certificate** button in Harbor to download the registry certificate.

10. **Deleting Repositories (Soft Deletion)**:
   - Deleting repositories involves two steps:
     - First, delete a repository in the Harbor interface. This is a soft deletion, meaning the repository is no longer managed by Harbor, but the repository files remain in the Harbor storage.
     - Next, run garbage collection in the Harbor interface to delete the repository files.

11. **Pulling Images from Harbor in Kubernetes (K8s)**:
   - If you're using Kubernetes, you can easily deploy pods with images stored in Harbor.
   - Modify the `daemon.json` on each worker node of your cluster if your Harbor instance is hosting HTTP and the certificate is self-signed.

Remember, these steps assume you've already set up Harbor and have a Docker image ready to push. Happy image pushing! 🚀🐳

Source: Conversation with Bing, 17/03/2024
(1) Harbor docs | Pulling and Pushing Images in the Docker Client. https://goharbor.io/docs/1.10/working-with-projects/working-with-images/pulling-pushing-images/.
(2) Get started with Harbor - Bitnami. https://docs.bitnami.com/aws/infrastructure/harbor/get-started/get-started/.
(3) Shipyard | GHA Recipes: Building Docker Images and Pushing to a .... https://shipyard.build/blog/gha-recipes-build-and-push-container-registry/.
(4) Pushing an Image into Harbor Image Registry | Know IT Like Pro. https://knowitlikepro.com/pushing-an-image-into-harbor/.

************** PUSHED WITHOUT SCANNING AND SIGNING THE IMAGE ********************
If you've already pushed the Docker image to Harbor and now want to delete it so you can rescan, sign, tag, and push the updated image, you can follow these steps:

1. **Delete the Docker Image from Harbor**:
   - Log in to your Harbor registry:
     ```bash
     docker login ethereum.gm-nig-ltd.tech
     ```
   - Use the `docker rmi` command to remove the image locally, and then push an empty image with the same tag to overwrite the existing image in Harbor. This essentially deletes the image from Harbor:
     ```bash
     docker rmi ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
     ```

2. **Scan and Sign the Image**:
   - After deleting the image, rescan it for vulnerabilities using a tool like Trivy or Clair, and sign it if necessary.

3. **Tag and Push the Updated Image**:
   - Once you're satisfied with the scanning and signing results, tag the image and push it to Harbor again:
     ```bash
     docker tag my-ethereum-app:latest ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
     docker push ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest
     ```

By following these steps, you can effectively delete the already pushed image from Harbor, rescan and sign it, and then push the updated version with the necessary security measures in place. Make sure to replace `my-ethereum-app` with your actual image name and `latest` with the appropriate tag.

************** SCANNING AND SIGNING ***********************
When dealing with container images, both **scanning** and **signing** play crucial roles in ensuring security. Let's explore the best practices for each step:

1. **Scanning Container Images**:
   - **Why?** Scanning is essential because it helps you find and fix vulnerabilities in container images **before** pushing them to your registry (such as Harbor, Docker Hub, ECR, or GCR).
   - **How?**
     - **CLI Scanning Locally**: Start by scanning locally using the Docker scan command. This allows you to catch security vulnerabilities before pushing any code. For example:
       ```
       docker scan <imageId or imageName>
       ```
       This command integrates with tools like Snyk to provide vulnerability reports.
     - **CI Pipeline Integration**: Embed scanning into your CI/CD pipeline. By doing this, you catch issues early and ensure a cleaner operating environment. Automate security scans at various stages of your pipeline to avoid incidents and save time.
     - **Cache Scan Results**: Use Docker's caching feature during image builds. Caching speeds up subsequent builds, but occasionally build images without cache to download security updates and OS patches.
     - **Scan Base Images**: New container images are often built off existing base images. Ensure you scan these base images for vulnerabilities¹.

2. **Signing Container Images**:
   - **Why?** Image signing ensures authenticity and integrity. It verifies that an image hasn't been tampered with.
   - **How?**
     - **Digital Signatures**: As an image builder, use a private key to sign the image. This creates a digital signature. To verify the signature, use the corresponding public key. For example:
       ```
       cosign verify --key cosign.pub <image-name>:<tag>
       ```
       Replace `<image-name>:<tag>` with your actual image details.
     - **Standard Practice**: While image signing methods vary across platforms, using a digital signature is common. It confirms the image's authenticity and prevents unauthorized modifications¹³.

In summary, **scan first** to catch vulnerabilities, and then **sign** to ensure authenticity. By following these practices, you'll build more secure containers and avoid security liabilities. 🛡️🐳

Source: Conversation with Bing, 17/03/2024
(1) 13 Container Image Scanning Best Practices & Examples. https://www.airplane.dev/blog/container-image-scanning.
(2) Sign and Verify Container Images with Docker, Cosign, and ... - Medium. https://medium.com/@seifeddinerajhi/sign-and-verify-container-images-with-cosign-and-kyverno-a-complete-guide-b32b1f6e6264.
(3) Docker Security Scanning Guide 2023 | Snyk. https://snyk.io/learn/docker-security-scanning/.
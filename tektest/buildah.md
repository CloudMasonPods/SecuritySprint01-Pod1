This follows a similar step as using kaniko for build and push , just using buildah which is a command-line utility and library for building Open Container Initiative (OCI) compatible container images. It enables users to create container images from scratch or modify existing container images without requiring a Docker daemon. Buildah focuses specifically on the build aspect of container management, providing a simple and scriptable way to build OCI-compliant images. It supports building images using Dockerfile instructions, enabling users to define the image's content and configuration. Additionally, Buildah allows users to push images to container registries, making it a versatile tool for managing the container image lifecycle. Buildah is often used in environments where a lightweight, daemonless approach to building container images is desired, offering flexibility and control over the container build process.The Open Container Initiative (OCI) is an open governance structure for the express purpose of creating open industry standards around container formats and runtime. Founded in 2015 by industry leaders like Docker, CoreOS, and Google, OCI aims to foster a vendor-neutral, community-driven approach to defining container specifications. At its core, OCI develops and maintains specifications for container runtime and image formats, ensuring interoperability and portability across different container platforms. This standardization facilitates smoother adoption and integration of container technologies across diverse environments, fostering innovation and collaboration within the container ecosystem
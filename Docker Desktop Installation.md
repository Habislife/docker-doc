Docker Desktop is free for personal use, learning purposes, and commercial use by teams with fewer than 250 employees and less than $10 million in annual revenue. Paid subscriptions are required for larger organizations.

## macOS Installation

1. **Visit the Docker Docs:** Go to [docs.docker.com/get-docker](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons/03-installation-and-set-up/docs.docker.com/get-docker) and click on "Docker Desktop for Mac" for either Intel or Apple M1, depending on your system.
2. **Download the installer:** The installer file (`.dmg`) will begin downloading.
3. **Install Docker Desktop:** Once downloaded, open the `.dmg` file and drag Docker into the Applications folder.
4. **Launch Docker Desktop:** Go to your Applications and click on Docker to start the application.

## Windows Installation

1. **Visit the Docker Docs:** Go to [docs.docker.com/get-docker](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons/03-installation-and-set-up/docs.docker.com/get-docker) and click on "Docker Desktop for Windows".
2. **Download the installer:** The installer file (`.exe`) will begin downloading.
3. **Install Docker Desktop:** Once downloaded, run the `.exe` file and follow the on-screen instructions. Choose either the WSL 2 backend (recommended) or Hyper-V as the backend system during the installation process.
4. **Launch Docker Desktop:** After installation, Docker Desktop should start automatically. If not, search for Docker in the Start menu and open the application.

## Linux Installation
1. **Visit the Docker Docs:** Go to [docs.docker.com/get-docker](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons/03-installation-and-set-up/docs.docker.com/get-docker) and click on "Docker Desktop for Linux".
2. **Install Using Package Manager:** Installation packages can be found for Debian, Fedora, Ubuntu, and Arch at [https://docs.docker.com/desktop/install/linux-install/#generic-installation-steps](https://docs.docker.com/desktop/install/linux-install/#generic-installation-steps)
3. **Use systemctl to launch Docker Desktop:** To start Docker Desktop use `systemctl --user start docker-desktop`

## Configuration and Resource Allocation

After installing Docker Desktop, you may want to adjust the allocated system resources:

1. **Open Docker Desktop settings:** Access the settings or preferences of the Docker Desktop application.
2. **Adjust resources:** Go to the "Resources" section and modify the CPU, memory, and disk space allocations as needed.
3. **Apply and restart:** Click "Apply and Restart" to save the changes and restart Docker Desktop.

With Docker Desktop installed, you can now run Docker commands with the CLI to communicate with the Docker daemon.
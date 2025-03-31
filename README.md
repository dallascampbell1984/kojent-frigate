<p align="center">
  <img align="center" alt="logo" src="docs/static/img/frigate.png">
</p>

---

> 🧠 **Kojent Frigate** is a user-friendly fork of Frigate NVR, designed to simplify setup, potentially eliminate unnecessary restarts for configuration changes (future goal), and make AI-powered surveillance accessible to everyone.
>
> * **Current Features:** Allows startup without cameras enabled, simplified Docker Compose setup via `.env` file.
> * **Future Goals:** Explore dynamic configuration reloading, enhanced UI configuration.

---

**Note:** This is the repository for **Kojent Frigate**, a fork of the official Frigate NVR.

**For instructions specific to setting up and running *Kojent Frigate*, please see the "[Kojent Frigate Quick Start](#kojent-frigate-quick-start)" section below.**

For original Frigate project documentation, please visit [https://docs.frigate.video](https://docs.frigate.video).

---

## 📚 Contents

* [Kojent Frigate Quick Start](#kojent-frigate-quick-start)
    * [1. Prepare Environment File](#1-prepare-environment-file)
    * [2. Prepare Host Directories (Only if using Bind Mounts)](#2-prepare-host-directories-only-if-using-bind-mounts)
    * [3. Choose Compose File & Deploy](#3-choose-compose-file--deploy)
    * [4. Access Web UI](#4-access-web-ui)
* [Volume Configuration Details (Named vs Bind Mounts)](#volume-configuration-details-named-vs-bind-mounts)
* [Official Documentation](#documentation)
* [Donations (Original Project)](#donations-original-project)
* [Screenshots (Original Project)](#screenshots-original-project)

---

# Frigate - NVR With Realtime Object Detection for IP Cameras

*(Original Frigate Introduction)*

[English] | [简体中文](https://github.com/blakeblackshear/frigate/README_CN.md)

A complete and local NVR designed for [Home Assistant](https://www.home-assistant.io) with AI object detection. Uses OpenCV and Tensorflow to perform realtime object detection locally for IP cameras.

Use of a [Google Coral Accelerator](https://coral.ai/products/) is optional, but highly recommended for object detection performance. The Coral will outperform even the best CPUs and can process 100+ FPS with very little overhead.

* Tight integration with Home Assistant via a [custom component](https://github.com/blakeblackshear/frigate-hass-integration)
* Designed to minimize resource use and maximize performance by only looking for objects when and where it is necessary
* Leverages multiprocessing heavily with an emphasis on realtime over processing every frame
* Uses a very low overhead motion detection to determine where to run object detection
* Object detection with TensorFlow (or other detectors) runs in separate processes for maximum FPS
* Communicates over MQTT for easy integration into other systems
* Records video with retention settings based on detected objects and/or motion
* 24/7 recording options
* Re-streaming via RTSP (using internal Go2RTC) to reduce the number of connections to your camera
* WebRTC & MSE support for low-latency live view in the UI

---

## Kojent Frigate Quick Start

This section explains how to get started with the Kojent Frigate fork using Docker Compose. Choose the setup that matches your needs.

> ⚠️ **Default Storage: Docker Named Volumes**
> By default, Kojent Frigate uses Docker Named Volumes for configuration (`frigate_config`) and storage (`frigate_storage`). This is the easiest method for GUI users (Portainer, Docker Desktop) as **no path editing is required** in the compose file itself. However, Docker manages the storage location (often on your OS drive), and accessing files directly is harder. See "[Volume Configuration](#volume-configuration-details-named-vs-bind-mounts)" below if you want to store recordings on a specific drive or access `config.yml` directly using Bind Mounts.

### 1. Prepare Environment File (`.env`)

* Copy the included `.env.example` file to a new file named `.env` in the same directory as your chosen compose file:
    ```bash
    cp .env.example .env
    ```
* **Edit the `.env` file** with a text editor. Review and customize all settings:
    * **Volume Configuration:** Decide between Docker Named Volumes (easy default) or Host Path Bind Mounts (advanced control). Edit the `CONFIG_VOLUME` and `MEDIA_VOLUME` variables according to the comments in the `.env` file. **This is the most important step for controlling where your data lives.**
    * **(Advanced Setup Only):** Fill in your correct `MQTT_HOST`, `MQTT_USER`, and `MQTT_PASSWORD`.
    * *(Optional):* Adjust `KOJENT_IMAGE`, `CONTAINER_NAME`, `WEB_PORT`, `SHM_SIZE`, `TZ` as needed.

### 2. Prepare Host Directories (Only if using Bind Mounts)

* **If, and only if,** you chose **Option 2 (Bind Mounts)** in your `.env` file by setting `CONFIG_VOLUME` and `MEDIA_VOLUME` to specific host paths (e.g., `/home/user/frigate/config`):
    * You **MUST ensure those directories exist** on your host machine before starting the container. Use the `mkdir -p` command if needed:
        ```bash
        # Example - Use the actual paths you set in .env!
        mkdir -p /path/on/your/host/to/frigate_config
        mkdir -p /path/on/your/host/to/frigate_storage
        ```
    * Place your Frigate `config.yml` file inside the host directory you specified for `CONFIG_VOLUME`.
    * If using the advanced compose file (`docker-compose-advanced.yml`), ensure the `mqtt:` section is removed or commented out in your `config.yml`, as MQTT settings will come from the `.env` file.

### 3. Choose Compose File & Deploy

* **Basic Setup (No MQTT/HA Integration):**
    * Uses `docker-compose.yml`.
    * Ensure your `.env` file is configured (especially volume choice).
    * Run from the directory containing the files:
        ```bash
        docker compose up -d
        # Use --build flag if you need to build the Kojent image locally:
        # docker compose up --build -d
        ```
* **Advanced Setup (With MQTT for Home Assistant):**
    * Uses `docker-compose-advanced.yml`.
    * Ensure your `.env` file has the correct MQTT details and volume choice configured.
    * Run from the directory containing the files:
        ```bash
        docker compose -f docker-compose-advanced.yml up -d
        # Use --build flag if building locally:
        # docker compose -f docker-compose-advanced.yml up --build -d
        ```
* **Using Portainer or Docker Desktop:**
    1.  Go to Stacks -> Add Stack (or equivalent).
    2.  Give the stack a name.
    3.  Choose the "Web editor" option.
    4.  Paste the entire content of either `docker-compose.yml` or `docker-compose-advanced.yml` into the editor.
    5.  Configure Environment Variables: Use Portainer/Docker Desktop's UI section for environment variables. Either point it to your edited `.env` file (if supported) or **manually add** each variable (`KOJENT_IMAGE`, `CONTAINER_NAME`, `WEB_PORT`, `SHM_SIZE`, `TZ`, `CONFIG_VOLUME`, `MEDIA_VOLUME`, and MQTT variables if using advanced) and set their values according to your `.env` file. **This step is crucial for overriding defaults.**
    6.  Click "Deploy the stack".

### 4. Access Web UI

* Once the container is running, access the Web UI at `http://<your-docker-host-ip>:5000` (or the port set via `WEB_PORT` in `.env`).
* If you used the default **Named Volume** for config (`CONFIG_VOLUME=frigate_config`), you will need to create your `config.yml` using the built-in editor in the Web UI for the first time.

---

### Volume Configuration Details (Named vs Bind Mounts)

The `.env` file controls how Frigate's configuration and media data are stored via the `CONFIG_VOLUME` and `MEDIA_VOLUME` variables. Choose **one** option in your `.env` file:

* **🟢 Option 1: Named Volumes (Default - Easiest)**
    * Set in `.env`:
        ```env
        CONFIG_VOLUME=frigate_config
        MEDIA_VOLUME=frigate_storage
        ```
    * **Pros:** Simplest setup, no need to specify host paths. Docker manages everything. Works immediately via GUI deployment.
    * **Cons:** Docker determines the storage location (usually on the OS drive). Recordings might fill up unintended drives. Direct access to `config.yml` and recordings from the host filesystem is more difficult (requires Docker commands or specific tools).

* **🛠️ Option 2: Bind Mounts (Advanced - Full Control)**
    * Set in `.env` (Example - **EDIT PATHS!**):
        ```env
        # Comment out or delete the Named Volume lines above
        CONFIG_VOLUME=/home/user/frigate/config  # Replace with your actual absolute path
        MEDIA_VOLUME=/mnt/data/frigate_media    # Replace with your actual absolute path
        ```
    * **Pros:** You have full control over where configuration and recordings are stored (e.g., use a large dedicated drive for media). Easy to access/edit `config.yml` and browse recordings directly on the host.
    * **Cons:** Requires you to edit the `.env` file with the correct **absolute paths** for *your* system. You must ensure these directories exist on your host *before* deploying.

---

## Documentation

View the **official Frigate documentation** at [https://docs.frigate.video](https://docs.frigate.video) for details on configuration options (`config.yml`), features, integrations, and more. Remember that some behaviors (like startup without cameras) might differ in this Kojent Frigate fork.

## Donations (Original Project)

If you would like to make a donation to support the development of the **original Frigate project**, please use [Github Sponsors](https://github.com/sponsors/blakeblackshear).

## Screenshots (Original Project)

*(Original Frigate Screenshots remain below)*

### Live dashboard
<div>
<img width="800" alt="Live dashboard" src="https://github.com/blakeblackshear/frigate/assets/569905/5e713cb9-9db5-41dc-947a-6937c3bc376e">
</div>

### Streamlined review workflow
<div>
<img width="800" alt="Streamlined review workflow" src="https://github.com/blakeblackshear/frigate/assets/569905/6fed96e8-3b18-40e5-9ddc-31e6f3c9f2ff">
</div>

### Multi-camera scrubbing
<div>
<img width="800" alt="Multi-camera scrubbing" src="https://github.com/blakeblackshear/frigate/assets/569905/d6788a15-0eeb-4427-a8d4-80b93cae3d74">
</div>

### Built-in mask and zone editor
<div>
<img width="800" alt="Multi-camera scrubbing" src="https://github.com/blakeblackshear/frigate/assets/569905/d7885fc3-bfe6-452f-b7d0-d957cb3e31f5">
</div>

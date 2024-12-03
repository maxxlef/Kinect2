# Installation and Configuration of libfreenect2 on Ubuntu 22.04

This guide explains how to set up **libfreenect2** on Ubuntu 22.04 to use with a Kinect Xbox One (Kinect v2). This setup is tailored for a system with an **NVIDIA RTX 4070** and an **AMD Ryzen** processor.

## 1. System Prerequisites

Make sure you have the NVIDIA drivers and CUDA installed to take full advantage of your GPU. Then, install the required development tools:

```bash
sudo apt-get update
sudo apt-get install build-essential cmake pkg-config libusb-1.0-0-dev libturbojpeg0-dev libglfw3-dev
```

## 2. Installation of libfreenect2

### Clone the Git repository and build it:

```bash
git clone https://github.com/OpenKinect/libfreenect2.git
cd libfreenect2
```

If necessary, install dependencies for older systems:

```bash
cd depends
./download_debs_trusty.sh
```

Then, return to the root project directory and build the project:

```bash
cd ..
mkdir build && cd build
cmake ..
make
sudo make install
```

## 3. Configure udev Rules for Kinect

Set up udev rules to allow access to the Kinect device without needing `sudo`:

```bash
sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Replug your Kinect device.

## 4. Testing the Kinect with Protonect

To verify the installation and check if the Kinect is detected, run the `Protonect` program from the `build` directory:

```bash
./bin/Protonect
```

## 5. Troubleshooting

If the Kinect is not detected:

1. Unplug and replug the Kinect device.
2. Verify that the udev rules are correctly applied by re-running:
   ```bash
   sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```

Once completed, try running `Protonect` again.

## Additional Resources

- [libfreenect2 GitHub Repository](https://github.com/OpenKinect/libfreenect2)

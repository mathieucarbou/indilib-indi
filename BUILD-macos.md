# Build INDI on macOS

## Prerequisites

```bash
# 1. Xcode Command Line Tools
xcode-select --install

# 2. Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 3. Dependencies
brew install cmake llvm libnova cfitsio libusb libev fftw gsl zlib curl wget ninja swig
```

> **Why LLVM?** Apple's `/usr/bin/c++` is missing C++ standard headers. Homebrew LLVM provides complete libc++.

## Build INDI Core

```bash
cd /path/to/indi
rm -rf build && mkdir build && cd build

cmake .. \
  -DCMAKE_C_COMPILER=/opt/homebrew/opt/llvm/bin/clang \
  -DCMAKE_CXX_COMPILER=/opt/homebrew/opt/llvm/bin/clang++ \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_FLAGS="-Wno-error=deprecated-declarations -Wno-error=unused-but-set-variable -Wno-error=unused-but-set-global" \
  -DCMAKE_C_FLAGS="-Wno-error=unused-but-set-variable -Wno-error=unused-but-set-global" \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=14.0

make -j$(sysctl -n hw.ncpu)
```

## Build INDI 3rd Party Drivers (ZWO ASI, etc.)

### Step 1: Install INDI core to a staging directory

```bash
cd /path/to/indi/build
DESTDIR=/tmp/indi-install make install
```

### Step 2: Build ASI SDK libraries

```bash
mkdir -p /tmp/indi-3rdparty-build && cd /tmp/indi-3rdparty-build

export INDI_ROOT=/tmp/indi-install/usr/local

cmake /path/to/indi-3rdparty \
  -DCMAKE_C_COMPILER=/opt/homebrew/opt/llvm/bin/clang \
  -DCMAKE_CXX_COMPILER=/opt/homebrew/opt/llvm/bin/clang++ \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_FLAGS="-Wno-error=deprecated-declarations -Wno-error=unused-but-set-variable -Wno-error=unused-but-set-global" \
  -DCMAKE_C_FLAGS="-Wno-error=unused-but-set-variable -Wno-error=unused-but-set-global" \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=14.0 \
  -DCMAKE_INSTALL_PREFIX=/tmp/indi-install/usr/local \
  -DFIX_MACOS_LIBS=ON \
  -DBUILD_LIBS=ON \
  -DWITH_ASICAM=ON \
  -DINDI_INSTALL_FIRMWARE=OFF \
  -DINDI_INSTALL_UDEV_RULES=OFF

make -j$(sysctl -n hw.ncpu)
make install
```

> **Important:** `-DCMAKE_OSX_ARCHITECTURES=arm64` is required on Apple Silicon.

### Step 3: Build ASI drivers

```bash
rm -rf /tmp/indi-3rdparty-build && mkdir -p /tmp/indi-3rdparty-build && cd /tmp/indi-3rdparty-build

export INDI_ROOT=/tmp/indi-install/usr/local

cmake /path/to/indi-3rdparty \
  -DCMAKE_C_COMPILER=/opt/homebrew/opt/llvm/bin/clang \
  -DCMAKE_CXX_COMPILER=/opt/homebrew/opt/llvm/bin/clang++ \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_FLAGS="-Wno-error=deprecated-declarations -Wno-error=unused-but-set-variable -Wno-error=unused-but-set-global" \
  -DCMAKE_C_FLAGS="-Wno-error=unused-but-set-variable -Wno-error=unused-but-set-global" \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=14.0 \
  -DCMAKE_INSTALL_PREFIX=/tmp/indi-install/usr/local \
  -DFIX_MACOS_LIBS=ON \
  -DBUILD_LIBS=OFF \
  -DWITH_ASICAM=ON \
  -DINDI_INSTALL_FIRMWARE=OFF \
  -DINDI_INSTALL_UDEV_RULES=OFF \
  -DWITH_EQMOD=OFF -DWITH_ASTARBOX=OFF -DWITH_STARBOOK=OFF -DWITH_STARBOOK_TEN=OFF \
  -DWITH_CAUX=OFF -DWITH_SX=OFF -DWITH_MAXDOME=OFF -DWITH_NEXDOME=OFF \
  -DWITH_SPECTRACYBER=OFF -DWITH_CLOUDWATCHER=OFF -DWITH_MI=OFF -DWITH_FLI=OFF \
  -DWITH_SBIG=OFF -DWITH_INOVAPLX=OFF -DWITH_APOGEE=OFF -DWITH_FFMV=OFF \
  -DWITH_QHY=OFF -DWITH_GPHOTO=OFF -DWITH_QSI=OFF -DWITH_DUINO=OFF \
  -DWITH_FISHCAMP=OFF -DWITH_GPSD=OFF -DWITH_DSI=OFF -DWITH_MGEN=OFF \
  -DWITH_ASTROMECHFOC=OFF -DWITH_LIMESDR=OFF -DWITH_RADIOSIM=OFF \
  -DWITH_GPSNMEA=OFF -DWITH_RTKLIB=OFF -DWITH_ARMADILLO=OFF \
  -DWITH_NIGHTSCAPE=OFF -DWITH_ATIK=OFF -DWITH_ATIK_EFW=OFF \
  -DWITH_TOUPCAM=OFF -DWITH_ALTAIRCAM=OFF -DWITH_BRESSERCAM=OFF \
  -DWITH_MALLINCAM=OFF -DWITH_MEADECAM=OFF -DWITH_NNCAM=OFF \
  -DWITH_OGMACAM=OFF -DWITH_OMEGONPROCAM=OFF -DWITH_STARSHOOTG=OFF \
  -DWITH_TSCAM=OFF -DWITH_SVBONYCAM=OFF -DWITH_DREAMFOCUSER=OFF \
  -DWITH_AVALON=OFF -DWITH_AVALONUD=OFF -DWITH_BEEFOCUS=OFF \
  -DWITH_SHELYAK=OFF -DWITH_SKYWALKER=OFF -DWITH_TALON6=OFF \
  -DWITH_PENTAX=OFF -DWITH_ASTROLINK4=OFF -DWITH_ORION_SSG3=OFF \
  -DWITH_SVBONY=OFF -DWITH_BRESSEREXOS2=OFF -DWITH_PLAYERONE=OFF \
  -DWITH_WEEWX_JSON=OFF -DWITH_ROLLOFFINO=OFF -DWITH_ASTROASIS=OFF \
  -DWITH_SCOPELINK=OFF -DWITH_OCS=OFF -DWITH_GPIO=OFF \
  -DWITH_TICFOCUSER-NG=OFF -DWITH_OPENOGMA=OFF -DWITH_WEBCAM=OFF -DWITH_NUT=OFF

make -j$(sysctl -n hw.ncpu)
```

### Step 4: Fix ASI library paths (macOS specific)

```bash
# Fix libusb reference in ASI SDK libraries
for lib in /tmp/indi-install/usr/local/lib/libASICamera2.dylib \
           /tmp/indi-install/usr/local/lib/libEFWFilter.dylib \
           /tmp/indi-install/usr/local/lib/libEAFFocuser.dylib \
           /tmp/indi-install/usr/local/lib/libCAARotator.dylib; do
  install_name_tool -change "@rpath/libusb-1.0.0.dylib" \
    "/opt/homebrew/opt/libusb/lib/libusb-1.0.0.dylib" "$lib"
done
```

### Step 5: Copy ASI drivers and libraries to INDI build

```bash
# Copy ASI SDK libraries
cp /tmp/indi-install/usr/local/lib/libASI*.dylib /path/to/indi/build/libs/indibase/
cp /tmp/indi-install/usr/local/lib/libEFW*.dylib /path/to/indi/build/libs/indibase/
cp /tmp/indi-install/usr/local/lib/libEAF*.dylib /path/to/indi/build/libs/indibase/
cp /tmp/indi-install/usr/local/lib/libCAA*.dylib /path/to/indi/build/libs/indibase/

# Fix libusb in copied libraries
for lib in /path/to/indi/build/libs/indibase/libASI*.dylib \
           /path/to/indi/build/libs/indibase/libEFW*.dylib \
           /path/to/indi/build/libs/indibase/libEAF*.dylib \
           /path/to/indi/build/libs/indibase/libCAA*.dylib; do
  install_name_tool -change "@rpath/libusb-1.0.0.dylib" \
    "/opt/homebrew/opt/libusb/lib/libusb-1.0.0.dylib" "$lib"
done

# Copy alignment library to indibase (ASI drivers need it)
cp /path/to/indi/build/libs/alignment/libindiAlignmentDriver*.dylib \
   /path/to/indi/build/libs/indibase/

# Copy ASI drivers
mkdir -p /path/to/indi/build/drivers/asi
cp /tmp/indi-3rdparty-build/indi-asi/indi_asi_* /path/to/indi/build/drivers/asi/

# Add rpaths to ASI drivers
for drv in /path/to/indi/build/drivers/asi/indi_asi_*; do
  install_name_tool -add_rpath /path/to/indi/build/libs/indibase "$drv" 2>/dev/null || true
  install_name_tool -add_rpath /path/to/indi/build/libs/alignment "$drv" 2>/dev/null || true
done
```

## Examples

### Example 1: Simulator only (no hardware)

```bash
cd /path/to/indi/build
./indiserver/indiserver -p 7624 \
  drivers/telescope/indi_simulator_telescope \
  drivers/ccd/indi_simulator_ccd \
  drivers/focuser/indi_simulator_focus
```

### Example 2: ZWO ASI camera

```bash
cd /path/to/indi/build
./indiserver/indiserver -p 7624 drivers/asi/indi_asi_ccd
```

### Example 3: Alpaca Server + ZWO ASI Camera

This exposes INDI devices via the ASCOM Alpaca protocol (HTTP on port 11111).

```bash
cd /path/to/indi/build

# Terminal 1: Start indiserver with Alpaca bridge and ASI camera
./indiserver/indiserver -p 7624 -v \
  drivers/alpaca/indi_alpaca_server \
  drivers/asi/indi_asi_ccd

# Terminal 2: Connect the Alpaca Server driver
./tools/indi_setprop -p 7624 "INDI Alpaca Server.CONNECTION.CONNECT=On"

# Wait 5 seconds for bridges to be created, then connect the camera
sleep 5
./tools/indi_setprop -p 7624 "ZWO CCD ASI676MM.CONNECTION.CONNECT=On"
```

**Test the Alpaca API:**

```bash
# Server info
curl http://localhost:11111/management/apiversions
# → {"Value":[1],"ErrorNumber":0,...}

curl http://localhost:11111/management/v1/description
# → {"Value":"INDI Alpaca Server","ErrorNumber":0,...}

# Camera info (device 0 = ASI676MM)
curl "http://localhost:11111/api/v1/camera/0/description?ClientID=1&ClientTransactionID=1"
# → {"Value":"INDI Camera Bridge for ZWO CCD ASI676MM","ErrorNumber":0,...}

# Check camera capabilities
curl "http://localhost:11111/api/v1/camera/0/canabortexposure?ClientID=1&ClientTransactionID=2"
# → {"Value":true,...}

curl "http://localhost:11111/api/v1/camera/0/camerastate?ClientID=1&ClientTransactionID=3"
# → {"Value":0,...}  (0=Idle, 1=Waiting, 2=Exposing, 3=Reading, 4=Error, 5=NotConnected)
```

> **Note:** The `connected` property may show `false` even when the camera is connected
> via INDI. This is a known issue with the Alpaca bridge's property sync. The camera
> is still usable — connect it via INDI first, then use Alpaca for control.

**Alpaca Discovery (UDP):**

Alpaca clients discover the server by broadcasting to UDP port 32227:

```python
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.settimeout(3)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
sock.sendto(b'alpacadiscovery1', ('255.255.255.255', 32227))
print(sock.recvfrom(1024)[0].decode())  # {"AlpacaPort":11111}
```

### Example 4: Alpaca Server + ASI Camera + Telescope

```bash
cd /path/to/indi/build

# Terminal 1
./indiserver/indiserver -p 7624 -v \
  drivers/alpaca/indi_alpaca_server \
  drivers/asi/indi_asi_ccd \
  drivers/telescope/indi_simulator_telescope

# Terminal 2
./tools/indi_setprop -p 7624 "INDI Alpaca Server.CONNECTION.CONNECT=On"
sleep 5
./tools/indi_setprop -p 7624 "ZWO CCD ASI676MM.CONNECTION.CONNECT=On"
./tools/indi_setprop -p 7624 "Telescope Simulator.CONNECTION.CONNECT=On"
```

**Test the Alpaca API:**

```bash
# Camera (device 0)
curl "http://localhost:11111/api/v1/camera/0/description?ClientID=1&ClientTransactionID=1"
# → "INDI Camera Bridge for ZWO CCD ASI676MM"

# Telescope (device 1)
curl "http://localhost:11111/api/v1/telescope/1/description?ClientID=1&ClientTransactionID=10"
# → "INDI Telescope: Telescope Simulator"

curl "http://localhost:11111/api/v1/telescope/1/canpark?ClientID=1&ClientTransactionID=11"
# → true
```

**Device mapping:**

| Alpaca Device | Type | INDI Device |
|--------------|------|-------------|
| `/api/v1/camera/0` | camera | ZWO CCD ASI676MM |
| `/api/v1/telescope/1` | telescope | Telescope Simulator |

### Example 5: Available ASI drivers

```bash
cd /path/to/indi/build

# ASI Camera (multi-camera)
./indiserver/indiserver -p 7624 drivers/asi/indi_asi_ccd

# ASI Camera (single camera)
./indiserver/indiserver -p 7624 drivers/asi/indi_asi_single_ccd

# ASI EAF Focuser
./indiserver/indiserver -p 7624 drivers/asi/indi_asi_focuser

# ASI EFW Filter Wheel
./indiserver/indiserver -p 7624 drivers/asi/indi_asi_wheel

# ASI CAA Rotator
./indiserver/indiserver -p 7624 drivers/asi/indi_asi_rotator
```

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `'string' file not found` | Wrong compiler | Use `/opt/homebrew/opt/llvm/bin/clang++` |
| `execlp: No such file or directory` | Driver not in PATH | Use full paths or `make install` |
| ASI driver crashes (exit 137) | Wrong libusb path | Run `install_name_tool` fix (Step 4) |
| ASI driver can't find `libindiAlignmentDriver` | Missing rpath | Copy lib to `libs/indibase/` (Step 5) |
| Alpaca API: "Device not found" | Bridges not created | Rebuild with the Alpaca fix patches |
| `building for macOS-14.0, but dylib built for newer` | Version mismatch | Warning only, safe to ignore |

## Alpaca Server Bug Fix (already applied)

The INDI Alpaca Server has a bug where `getDriverInterface()` returns 0 because
C++ `PropertyText` widgets are empty when `newProperty()` is called. Fixed by:

- **`drivers/alpaca/alpaca_client.cpp`**: Extract `DRIVER_INTERFACE` from raw C struct
- **`drivers/alpaca/device_manager.cpp/h`**: Added `forceBridgeCreation()` method
- **`drivers/alpaca/indi_alpaca_server.cpp`**: Delayed retry after client connects

> **Known limitation:** The `connected` property may not sync correctly between
> INDI and Alpaca (same C++ widget bug). Connect devices via INDI first, then
> use Alpaca for API access.

## Resources

- [INDI](https://indilib.org) | [Docs](https://docs.indilib.org) | [GitHub](https://github.com/indilib/indi)
- [ASCOM Alpaca API](https://ascom-standards.org/api/)

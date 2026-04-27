# proton_vendor

Proton Vendor package that is used to package [Proton](https://github.com/clearpathrobotics/proton) as a ROS 2 package.

## Building

By default, this package downloads and builds Proton from GitHub. For development with local Proton changes, you can use Proton from the same workspace:

```
colcon build --cmake-args -DPROTON_VENDOR_USE_LOCAL=ON --symlink-install
```

Ensure Proton is cloned in the same workspace when using this option.

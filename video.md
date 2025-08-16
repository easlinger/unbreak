# Useful Commands

`nvidia-smi` or `watch -n 1 nvidia-smi` for live

# General Troubleshooting

Ensure in video user groups (see output of `groups` command)

```
sudo usermod -aG video $USER
sudo usermod -aG render $USER
```

# Drivers

Get them from here: https://github.com/NVIDIA/open-gpu-kernel-modules

Purge all nvidia* first.

If you're installing using `chroot`, might need to alter make files/commands so kernel versions are pulled from your actual ones rather than the boot drive.

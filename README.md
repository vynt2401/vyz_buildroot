# vyz_buildroot (Custom Linux Distribution Project)

[![Embedded Linux](https://img.shields.io/badge/Platform-Embedded%20Linux-orange.svg)](https://www.buildroot.org)
[![Buildroot](https://img.shields.io/badge/Tool-Buildroot%202023.x%2F2024.x-blue.svg)](https://buildroot.org)
[![Target](https://img.shields.io/badge/Target-QEMU%20VExpress%20%2F%20OrangePi%20Zero-green.svg)]()

Dự án này sử dụng công cụ **Buildroot** để cấu hình, biên dịch chéo (cross-compile) và xây dựng một hệ điều hành Linux nhúng tùy biến nhẹ, tối ưu (Custom Embedded Linux Distribution). Hệ thống được cấu hình để chạy giả lập trên môi trường QEMU (ARM Versatile Express) hoặc triển khai trực tiếp trên phần cứng máy tính nhúng (SBC).

---

## 📌 Các Tính Năng Chính
- **Toolchain:** Sử dụng bộ dịch chéo (Cross-compilation Toolchain) tối ưu cho kiến trúc ARM.
- **Kernel:** Tùy biến Linux Kernel, lược bỏ các driver không cần thiết để giảm dung lượng và tăng tốc độ boot.
- **Root Filesystem (RootFS):** Tích hợp công cụ tiện ích hệ thống qua BusyBox.
- **Hỗ trợ mạng:** Cấu hình Network Programming (TCP/IP), SSH server, Dropbear nhằm phục vụ việc giao tiếp và quản lý từ xa.
- **Quản lý dữ liệu:** Hỗ trợ SQLite cho các tác vụ lưu trữ lịch sử offline trực tiếp trên board.

---

## 📂 Cấu Trúc Thư Mục Dự Án

text
vyz_buildroot/
├── buildroot-vexpress/       # Mã nguồn Buildroot chính (Sub-directory)
│   ├── board/                # Cấu hình đặc thù cho bo mạch (QEMU / Orange Pi)
│   ├── configs/              # File cấu hình mặc định (defconfig) của dự án
│   │   └── vyz_vexpress_defconfig
│   ├── output/               # [Bị ẩn bởi .gitignore] Thư mục chứa sản phẩm sau khi build
│   │   ├── images/           # Chứa zImage, rootfs.tar, vexpress-v2p-ca9.dtb
│   │   └── build/            # Thư mục tạm trong quá trình biên dịch
│   └── .config               # File cấu hình Buildroot hiện tại
├── scripts/                  # Các script tự động hóa (QEMU boot, flash SD card)
└── README.md                 # Tài liệu hướng dẫn này

## Hướng Dẫn Cài Đặt & Biên Dịch ##
1. Chuẩn bị môi trường (Host OS)
Khuyến khích sử dụng hệ điều hành Fedora Linux hoặc Ubuntu/Debian thông qua WSL2 trên Windows 11. Cài đặt các gói phụ thuộc cần thiết cho Buildroot:
```
# Đối với Ubuntu/Debian/WSL:
sudo apt-get install cpio unzip rsync make gcc g++ bc ncurses-dev

# Đối với Fedora:
sudo dnf groupinstall "Development Tools" "Development Libraries"
sudo dnf install ncurses-devel bc rsync cpio unzip
```

2. Cấu hình hệ thống
Di chuyển vào thư mục Buildroot và nạp cấu hình mặc định của dự án:

```
cd buildroot-vexpress
make vyz_vexpress_defconfig
Nếu bạn muốn thay đổi cấu hình hệ thống, nhân Kernel hoặc BusyBox:

Bash
make menuconfig       # Cấu hình các gói phần mềm hệ thống
make linux-menuconfig # Cấu hình tùy biến Linux Kernel
```

3. Tiến hành Biên Dịch (Build)
Quá trình build lần đầu sẽ tải mã nguồn từ internet về và biên dịch chéo, có thể mất từ 30 phút đến 1 tiếng tùy cấu hình phần cứng máy Host:

```
make -j$(nproc)
```

Sản phẩm sau khi biên dịch thành công nằm trong thư mục output/images/ bao gồm:

zImage: Linux Kernel image.

rootfs.ext2 / rootfs.tar: Hệ thống file gốc.

vexpress-v2p-ca9.dtb: Device Tree Blob cho bo mạch giả lập.

### Hướng Dẫn Chạy Giả Lập (QEMU)
Để kiểm tra hệ điều hành vừa build mà không cần phần cứng thật, bạn có thể khởi chạy thông qua QEMU ARM:

```
qemu-system-arm -M versatilepb \
    -kernel output/images/zImage \
    -dtb output/images/vexpress-v2p-ca9.dtb \
    -drive file=output/images/rootfs.ext2,if=scsi,format=raw \
    -append "root=/dev/sda console=ttyAMA0,115200" \
    -net nic -net user \
    -nographic
```
Nhấn Ctrl + A rồi thả ra, sau đó nhấn X nếu muốn thoát khỏi màn hình giả lập QEMU.




# Các bước thực hiện chi tiết
## Bước 1: Kết nối vào Droplet bằng tài khoản root
1. Mở Terminal trên máy cá nhân, kết nối tới Droplet:
`ssh -i ~/.ssh/id_ed25519 root@<IP_PUBLIC_CUA_DROPLET>`
2. Xác nhận bạn đang đăng nhập với tư cách root bằng lệnh:
`whoami`
   Kết quả hiển thị root.
## Bước 2: Tạo tài khoản người dùng mới devops
1. Sử dụng lệnh adduser để tạo user mới tên devops maychuvina dataonline:
`adduser devops`
2. Hệ thống sẽ yêu cầu:
*    Nhập mật khẩu mới cho user, sau đó nhập lại để xác nhận (bắt buộc, mật khẩu phải đủ mạnh)
*    Nhập các thông tin bổ sung như Full Name, Room Number, Work Phone, Home Phone, Other - nhấn Enter để bỏ qua
*    Xác nhận thông tin bằng cách gõ Y rồi nhấn Enter

3. Kết quả khi tạo thành công:
```aiignore
Adding user `devops' ...
Adding new group `devops' (1001) ...
Adding new user `devops' (1001) with group `devops' ...
Creating home directory `/home/devops' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for devops
Enter the new value, or press ENTER for the default
Full Name []:
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] Y
```

## Bước 3: Thêm user devops vào nhóm quản trị sudo
1. Sử dụng lệnh usermod để thêm user vào nhóm sudo:
`usermod -aG sudo devops`
2. Giải thích quan trọng: Theo mặc định trên Ubuntu, tất cả các thành viên của nhóm sudo đều có đầy đủ quyền sudo - không
cần chỉnh sửa trực tiếp file /etc/sudoers. Tùy chọn -aG (append, Group) an toàn hơn, chỉ thêm user vào nhóm bổ sung mà không
gỡ bỏ các nhóm hiện có

## Bước 4: Sao chép cấu hình SSH Key từ thư mục root sang user devops
Để user devops đăng nhập được qua SSH bằng cùng khóa cá nhân, cần sao chép toàn bộ thư mục .ssh từ /root sang /home/devops, 
đồng thời tự động cập nhật owner sang devops:devops:
`sudo rsync --archive --chown=devops:devops ~/.ssh /home/devops/`

Giải thích lệnh:
* rsync: công cụ đồng bộ file an toàn, không làm hỏng cấu trúc thư mục
* --archive: sao chép giữ nguyên toàn bộ thuộc tính file (quyền, thời gian, symlink, ...)
* --chown=devops:devops: tự động đổi chủ sở hữu thành user devops và nhóm devops

## Bước 5: Phân quyền chính xác cho thư mục .ssh và file authorized_keys
Để cơ chế bảo mật SSH hoạt động đúng, cần thiết lập quyền hạn chế như sau:
`chmod 700 /home/devops/.ssh
chmod 600 /home/devops/.ssh/authorized_keys
chmod 644 /home/devops/.ssh/id_ed25519.pub`

Giải thích các quyền:
* 700: chỉ chủ sở hữu có quyền đọc/ghi/thực thi trong thư mục .ssh
* 600: chỉ chủ sở hữu có quyền đọc/ghi file authorized_keys
* 644: chủ sở hữu đọc/ghi, nhóm và người khác chỉ đọc - áp dụng cho file khóa công khai .pub

## Bước 6: Kiểm tra cấu hình SSH Key
1. Xác nhận file authorized_keys có nội dung khớp với khóa công khai ban đầu:
`cat /home/devops/.ssh/authorized_keys`
2. Đảm bảo chủ sở hữu của thư mục .ssh là devops:
`ls -la /home/devops/.ssh`

## Bước 7: Kiểm tra đăng nhập thành công bằng user devops
Từ máy cá nhân, mở cửa sổ Terminal mới và đăng nhập bằng user devops:
`ssh -i /path/to/private_key devops@<IP_ADDRESS_DROPLET>`
Ví dụ thực tế: `ssh -i ~/.ssh/id_ed25519 devops@157.240.23.100`
Nếu kết nối thành công, prompt sẽ hiển thị devops@hostname:~$.

## Bước 8: Kiểm tra đặc quyền sudo
1. Chạy lệnh kiểm tra:
`sudo whoami`
2. Hệ thống sẽ yêu cầu nhập mật khẩu của user devops (lần đầu sử dụng sudo)
3. Nếu thành công, kết quả hiển thị:
`root`
4. Có thể kiểm tra thêm bằng lệnh cập nhật thử nghiệm:
`sudo apt update`
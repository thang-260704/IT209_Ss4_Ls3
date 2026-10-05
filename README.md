# Bài 3: Cấu hình xác thực SSH và Đẩy dự án lên GitHub

## Mục tiêu
- Dùng cặp khóa SSH **Ed25519** để xác thực với GitHub.
- Liên kết repository cục bộ với GitHub qua **giao thức SSH** (không dùng HTTPS).
- Push mã nguồn và toàn bộ lịch sử commit lên GitHub.

## Kết quả

| Thông tin | Giá trị |
|---|---|
| Tài khoản GitHub | `thang-260704` |
| Repository | **https://github.com/thang-260704/IT209_Ss4_Ls2** |
| Remote URL (SSH) | `git@github.com:thang-260704/IT209_Ss4_Ls2.git` |
| Thuật toán khóa | Ed25519 |
| Dự án được push | Repo thực hành bài 2 (`/d/rikkei/IT209/Ss4/Ls2`), gồm cả lịch sử merge |

## Môi trường
- Windows 10, Git Bash (MINGW64)

---

## Các bước thực hiện

### Bước 1 – Tạo cặp khóa SSH Ed25519

```bash
ssh-keygen -t ed25519 -C "phanhtran0501@gmail.com"
```

Nhấn Enter để giữ đường dẫn mặc định. Lệnh tạo ra 2 file:

| File | Vai trò | Có chia sẻ không? |
|---|---|---|
| `~/.ssh/id_ed25519` | **Private key**: dùng để chứng minh danh tính | ❌ Tuyệt đối không chia sẻ, không commit |
| `~/.ssh/id_ed25519.pub` | **Public key**: đưa lên GitHub | ✅ |

> Cặp khóa này đã được tạo ở Session 02 (dùng để SSH vào VPS) và được dùng lại cho GitHub. Một public key có thể được thêm vào nhiều máy chủ hoặc dịch vụ khác nhau.

**Vì sao chọn Ed25519?** So với RSA, Ed25519 cho khóa ngắn hơn (68 ký tự so với hơn 500 ký tự), ký và xác thực nhanh hơn, độ an toàn tương đương RSA-3072 trở lên. Đây cũng là thuật toán GitHub khuyến nghị.

### Bước 2 – Thêm Public Key vào GitHub

```bash
cat ~/.ssh/id_ed25519.pub
```

1. Copy toàn bộ dòng kết quả, bắt đầu bằng `ssh-ed25519 ...`.
2. Trên GitHub, vào **Settings → SSH and GPG keys → New SSH key**.
3. Điền **Title** là `Laptop-ADMIN`, **Key type** là `Authentication Key`, dán public key vào ô **Key**, rồi bấm **Add SSH key**.

![SSH key trên GitHub](images/01-github-ssh-key.png)

### Bước 3 – Kiểm tra kết nối SSH tới GitHub

```bash
ssh -T git@github.com
```

Lần đầu kết nối, gõ `yes` để lưu fingerprint của GitHub vào `~/.ssh/known_hosts`. Kết quả:

```text
Hi thang-260704! You've successfully authenticated, but GitHub does not provide shell access.
```

GitHub đã nhận diện đúng tài khoản `thang-260704` qua SSH key. Câu "does not provide shell access" là bình thường: GitHub chỉ cho phép thao tác Git qua SSH, không cấp shell.

### Bước 4 – Chuyển remote sang URL SSH và push

```bash
cd /d/rikkei/IT209/Ss4/Ls2
git remote -v                    # trước: origin đang dùng HTTPS
git remote set-url origin git@github.com:thang-260704/IT209_Ss4_Ls2.git
git remote -v                    # sau: origin đã chuyển sang SSH
git push -u origin main
git push origin feature-update
```

- Repo này trước đó đã có `origin` dùng **HTTPS** (`https://github.com/...`). Lệnh `git remote set-url` đổi URL của `origin` sang dạng **SSH** (`git@github.com:...`) mà không cần xoá và tạo lại remote.
- `-u` (upstream) liên kết nhánh `main` cục bộ với `origin/main`. Từ lần sau chỉ cần gõ `git push` hoặc `git pull`.
- Không bị hỏi username, password hay token, vì xác thực diễn ra hoàn toàn qua SSH key.

Kết quả `git remote -v`:

```text
origin  git@github.com:thang-260704/IT209_Ss4_Ls2.git (fetch)
origin  git@github.com:thang-260704/IT209_Ss4_Ls2.git (push)
```

Log thực tế:

```text
$ git remote set-url origin git@github.com:thang-260704/IT209_Ss4_Ls2.git

$ git remote -v
origin  git@github.com:thang-260704/IT209_Ss4_Ls2.git (fetch)
origin  git@github.com:thang-260704/IT209_Ss4_Ls2.git (push)

$ ssh -T git@github.com
Hi thang-260704! You've successfully authenticated, but GitHub does not provide shell access.

$ git push -u origin main
branch 'main' set up to track 'origin/main'.
Everything up-to-date
```

Lệnh push chạy qua SSH mà **không bị hỏi username, password hay token**. Kết quả `Everything up-to-date` cho biết toàn bộ lịch sử commit (gồm cả merge commit của bài 2) đã có trên GitHub.

![ssh -T, git remote -v và git push](images/02-ssh-test-push.png)

---

## So sánh SSH và HTTPS khi làm việc với GitHub

| Tiêu chí | HTTPS | SSH |
|---|---|---|
| Dạng URL | `https://github.com/user/repo.git` | `git@github.com:user/repo.git` |
| Xác thực | Username + Personal Access Token | Cặp khóa public/private |
| Nhập thông tin mỗi lần push | Có (trừ khi dùng credential manager) | Không |
| Thông tin bí mật truyền qua mạng | Token | Không truyền private key. Chỉ gửi chữ ký số để chứng minh có key |

## Đối chiếu yêu cầu

| Yêu cầu | Kết quả |
|---|---|
| Sinh khóa bằng thuật toán **Ed25519** | ✅ `ssh-keygen -t ed25519` |
| `ssh -T git@github.com` xác nhận đúng tài khoản | ✅ `Hi thang-260704! You've successfully authenticated` |
| Remote origin có dạng `git@github.com:username/repository.git` | ✅ |
| Push mã nguồn và lịch sử commit lên GitHub qua SSH | ✅ |
| Không nộp private key | ✅ Chỉ public key được đưa lên GitHub |

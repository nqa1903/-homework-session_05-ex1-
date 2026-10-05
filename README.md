# Bài 1: Khôi phục commit đã mất bằng Git Reflog

## 1. Mục tiêu

- Hiểu cách Git lưu lịch sử hoạt động cục bộ thông qua **reflog**.
- Dùng `git reflog` để truy vết các commit cũ.
- Khôi phục commit bị mất sau `git reset --hard`, **không viết lại code thủ công**.

## 2. Các bước thực hiện

### Bước 1: Khởi tạo repository thử nghiệm và tạo commit ban đầu

```bash
mkdir reflog-demo && cd reflog-demo
git init
echo "Khoi tao du an" > README.txt
git add .
git commit -m "Khoi tao du an"
```

### Bước 2: Tạo commit quan trọng (commit sẽ bị mất)

```bash
echo "Day la tinh nang quan trong" > feature.txt
git add .
git commit -m "Them tinh nang quan trong"
git log --oneline
```

Kết quả:

```
2ed81ed Them tinh nang quan trong
1da1176 Khoi tao du an
```

### Bước 3: Giả lập sự cố - vô tình chạy `git reset --hard HEAD~1`

```bash
git reset --hard HEAD~1
```

Kết quả:

```
HEAD is now at 1da1176 Khoi tao du an
```

### Bước 4: Phát hiện commit đã mất

```bash
git log --oneline
ls
```

Kết quả:

```
1da1176 Khoi tao du an
```

```
README.txt
```

- Commit `Them tinh nang quan trong` **không còn** trong `git log`.
- File `feature.txt` **biến mất** khỏi thư mục làm việc (do `--hard` xóa luôn working directory).

### Bước 5: Tra cứu `git reflog` để tìm hash của commit đã mất

```bash
git reflog
```

Kết quả:

```
1da1176 HEAD@{0}: reset: moving to HEAD~1
2ed81ed HEAD@{1}: commit: Them tinh nang quan trong
1da1176 HEAD@{2}: commit (initial): Khoi tao du an
```

Dòng `2ed81ed HEAD@{1}: commit: Them tinh nang quan trong` cho biết commit cần khôi phục có hash là **`2ed81ed`**.

### Bước 6: Khôi phục commit bằng `git reset --hard <hash>`

```bash
git reset --hard 2ed81ed
```

Kết quả:

```
HEAD is now at 2ed81ed Them tinh nang quan trong
```

## 3. Kiểm tra kết quả

```bash
git log --oneline
```

```
2ed81ed Them tinh nang quan trong
1da1176 Khoi tao du an
```

```bash
ls
cat feature.txt
```

```
README.txt
feature.txt
Day la tinh nang quan trong
```

```bash
git reflog
```

```
2ed81ed HEAD@{0}: reset: moving to 2ed81ed
1da1176 HEAD@{1}: reset: moving to HEAD~1
2ed81ed HEAD@{2}: commit: Them tinh nang quan trong
1da1176 HEAD@{3}: commit (initial): Khoi tao du an
```

Commit `Them tinh nang quan trong` đã quay lại đúng tiêu đề, đúng hash gốc (`2ed81ed`), và file `feature.txt` với nội dung `Day la tinh nang quan trong` đã được khôi phục.

## 4. Giải thích

- **Reflog** là nhật ký cục bộ ghi lại mọi lần `HEAD` (và các nhánh) di chuyển: commit, reset, checkout, merge, rebase...
- `git reset --hard HEAD~1` chỉ làm nhánh hiện tại **trỏ lùi** về commit cha. Commit cũ không bị xóa ngay, chỉ là không còn nhánh nào trỏ tới nên `git log` không hiển thị.
- Vì reflog vẫn nhớ hash của commit đó, ta có thể trỏ nhánh về lại bằng `git reset --hard <hash>`.

### Cách khác: dùng checkout / branch

Nếu không muốn đổi nhánh hiện tại, có thể tạo nhánh mới tại commit đã mất:

```bash
git checkout -b recovered 2ed81ed
```

## 5. Lưu ý

- Reflog chỉ tồn tại **cục bộ**, không được push lên remote.
- Các mục reflog mặc định được giữ khoảng 90 ngày (mục còn truy cập được) hoặc 30 ngày (mục không còn truy cập được), sau đó có thể bị `git gc` dọn đi.
- Nên cẩn thận với `git reset --hard` vì nó xóa luôn thay đổi chưa commit; những thay đổi chưa từng được commit thì reflog **không** cứu được.

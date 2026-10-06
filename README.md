# Bài 5: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu

* Thực hành sử dụng `git reset` để quay lại trạng thái commit trước đó khi làm việc cục bộ.
* Thực hành sử dụng `git revert` để đảo ngược một commit mà không làm thay đổi lịch sử đã công khai.
* Phân biệt sự khác nhau giữa Reset và Revert.
* Hiểu cách lựa chọn Reset hoặc Revert khi làm việc cá nhân và làm việc nhóm.

## 2. Trường hợp 1: Sử dụng Reset

### Bước 1: Tạo commit ban đầu

Tạo file `code.txt` với nội dung:

```text
version 1
```

Sau đó thực hiện:

```bash
git add code.txt
git commit -m "add initial code"
```

### Bước 2: Tạo commit chứa code lỗi

Sửa `code.txt` và thêm nội dung:

```text
version 1
code loi
```

Sau đó commit:

```bash
git add code.txt
git commit -m "add buggy code"
```

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Kết quả ví dụ:

```text
b222222 add buggy code
a111111 add initial code
```

### Bước 3: Reset về commit trước

Sử dụng:

```bash
git reset --mixed HEAD~1
```

Không sử dụng:

```bash
git reset --hard HEAD~1
```

vì `--hard` có thể làm mất các thay đổi trong Working Directory.

Sau khi reset, kiểm tra:

```bash
git status
```

Kết quả cho thấy file `code.txt` ở trạng thái:

```text
modified: code.txt
```

Điều này chứng minh thay đổi của commit `add buggy code` vẫn được giữ lại trong Working Directory.

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Commit `add buggy code` không còn nằm trên nhánh local, trong khi nội dung thay đổi của file vẫn được giữ lại.

## 3. Trường hợp 2: Sử dụng Revert

Sau khi hoàn thành phần Reset, tạo các commit tiếp theo:

```bash
git add code.txt
git commit -m "add version 2"

git add code.txt
git commit -m "add version 3"
```

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Ví dụ:

```text
c333333 add version 3
b222222 add version 2
a111111 add initial code
```

Giả sử commit:

```text
b222222 add version 2
```

chứa thay đổi không mong muốn.

Sử dụng:

```bash
git revert b222222
```

Hoặc:

```bash
git revert --no-edit b222222
```

Git sẽ tạo một commit mới để đảo ngược thay đổi của commit được chọn.

Kiểm tra:

```bash
git log --oneline -n 5
```

Kết quả mong đợi:

```text
d444444 Revert "add version 2"
c333333 add version 3
b222222 add version 2
a111111 add initial code
```

Commit `Revert "add version 2"` được tạo mới và nằm trên đầu nhánh.

Commit `add version 2` vẫn được giữ nguyên trong lịch sử.

## 4. So sánh Git Reset và Git Revert

### 4.1. Điểm giống nhau

Cả `git reset` và `git revert` đều có thể được sử dụng để xử lý khi một commit chứa thay đổi không mong muốn.

Mục đích chung là đưa mã nguồn về trạng thái mong muốn, nhưng cách chúng xử lý lịch sử Git hoàn toàn khác nhau.

### 4.2. Bảng so sánh

| Tiêu chí                             | Git Reset                                          | Git Revert                                          |
| ------------------------------------ | -------------------------------------------------- | --------------------------------------------------- |
| Mục đích                             | Di chuyển HEAD về commit khác                      | Tạo commit mới để đảo ngược commit cũ               |
| Có tạo commit mới không?             | Không                                              | Có                                                  |
| Commit cũ                            | Có thể bị loại khỏi lịch sử của nhánh              | Vẫn được giữ trong lịch sử                          |
| Lịch sử Git                          | Có thể bị thay đổi                                 | Được giữ nguyên                                     |
| HEAD                                 | Di chuyển về commit trước                          | Vẫn tiến về phía trước                              |
| Có giữ thay đổi với `--mixed`?       | Có, giữ trong Working Directory                    | Không theo cơ chế reset; Git tạo thay đổi đảo ngược |
| Phù hợp làm việc cá nhân             | Rất phù hợp                                        | Có thể sử dụng                                      |
| Phù hợp nhánh chung                  | Không nên dùng để viết lại lịch sử đã chia sẻ      | Rất phù hợp                                         |
| Đã push lên remote                   | Không nên reset rồi force push nếu không cần thiết | Có thể revert an toàn                               |
| Rủi ro làm ảnh hưởng thành viên khác | Cao nếu viết lại lịch sử chung                     | Thấp hơn                                            |
| Cách xử lý lịch sử                   | Viết lại lịch sử                                   | Ghi thêm lịch sử mới                                |

## 5. Git Reset khi làm việc cá nhân

Khi làm việc cá nhân trên máy local và commit chưa được chia sẻ lên remote, `git reset` rất hữu ích.

Ví dụ:

```text
A → B → C
```

Trong đó `C` là commit bị lỗi.

Sử dụng:

```bash
git reset --mixed HEAD~1
```

Lịch sử trở thành:

```text
A → B
```

HEAD quay về `B`.

Tuy nhiên, các thay đổi từ `C` vẫn được giữ trong Working Directory ở trạng thái chưa staged.

Điều này cho phép lập trình viên sửa lại code và commit lại theo cách phù hợp.

## 6. Git Revert khi làm việc nhóm

Khi commit đã được push lên remote hoặc đã nằm trên nhánh chung, không nên tùy tiện sử dụng `git reset` để viết lại lịch sử.

Ví dụ:

```text
A → B → C
```

Nếu `B` chứa lỗi, sử dụng:

```bash
git revert <commit-B>
```

Git tạo thêm commit:

```text
A → B → C → D
```

Trong đó:

```text
D = Revert B
```

Commit `B` vẫn tồn tại trong lịch sử.

Cách này an toàn hơn khi làm việc nhóm vì các thành viên khác vẫn có thể đồng bộ lịch sử Git mà không phải xử lý việc lịch sử bị thay đổi.

## 7. Khi nào nên dùng Reset?

Nên sử dụng `git reset` khi:

* Đang làm việc cá nhân.
* Commit chưa được push lên remote.
* Muốn chỉnh sửa hoặc viết lại lịch sử commit.
* Muốn quay lại commit trước và tiếp tục phát triển từ đó.
* Muốn giữ lại thay đổi trong Working Directory bằng `--mixed`.

Ví dụ:

```bash
git reset --mixed HEAD~1
```

Không nên tùy tiện dùng reset trên nhánh chung đã được nhiều người cùng làm việc.

## 8. Khi nào nên dùng Revert?

Nên sử dụng `git revert` khi:

* Commit đã được push lên remote.
* Commit nằm trên nhánh chung.
* Nhiều thành viên đang làm việc trên cùng một nhánh.
* Muốn đảo ngược một commit nhưng vẫn giữ lịch sử rõ ràng.
* Muốn tạo một commit mới ghi nhận việc sửa lỗi.

Ví dụ:

```bash
git revert <commit-id>
```

## 9. Ví dụ minh họa sự khác nhau

### Reset

Lịch sử ban đầu:

```text
A → B → C
```

Sau:

```bash
git reset --mixed HEAD~1
```

Lịch sử:

```text
A → B
```

Commit `C` không còn nằm trên nhánh hiện tại.

Các thay đổi của `C` được giữ lại trong Working Directory.

### Revert

Lịch sử ban đầu:

```text
A → B → C
```

Sau:

```bash
git revert B
```

Lịch sử:

```text
A → B → C → D
```

Trong đó:

```text
D = Revert B
```

Commit `B` vẫn tồn tại và `D` là commit mới đảo ngược thay đổi của `B`.

## 10. Kết luận

`git reset` và `git revert` đều có thể được sử dụng để xử lý commit có lỗi nhưng phục vụ hai mục đích khác nhau.

`git reset` phù hợp với việc chỉnh sửa lịch sử cục bộ, đặc biệt khi làm việc cá nhân và commit chưa được chia sẻ. Với `git reset --mixed`, HEAD được đưa về commit trước nhưng các thay đổi vẫn được giữ trong Working Directory để tiếp tục chỉnh sửa.

`git revert` phù hợp với môi trường làm việc nhóm và các commit đã được push lên remote. Revert không xóa hoặc viết lại commit cũ mà tạo một commit mới để đảo ngược thay đổi. Vì vậy lịch sử Git vẫn rõ ràng và an toàn hơn khi nhiều thành viên cùng làm việc.

Có thể ghi nhớ ngắn gọn:

```text
RESET  → quay lại và viết lại lịch sử
REVERT → tạo commit mới để đảo ngược lịch sử
```

## 11. Các lệnh chính đã sử dụng

### Reset

```bash
git reset --mixed HEAD~1
git status
git log --oneline -n 5
```

### Revert

```bash
git revert <commit-id>
git revert --no-edit <commit-id>
git log --oneline -n 5
git status
```

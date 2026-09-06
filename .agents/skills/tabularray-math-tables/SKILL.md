---
name: tabularray-math-tables
description: "Dựng bảng toán học bằng tabularray theo đúng semantic hàng-cột, dùng kích thước vừa đủ và kế thừa style chung của repo; chỉ override cục bộ khi bảng nguồn thật sự khác."
metadata:
  language: "vi"
  version: "1.1.0"
  scope: "latex,tabularray,tblr,longtblr,tables,worksheet,math,layout"
---

# Bảng toán học với `tabularray`

Dùng skill này khi nội dung có **quan hệ hàng-cột thực sự**: bảng giá trị, số liệu, phân loại, so sánh, bảng điền, Đúng/Sai, ghép cặp hoặc bảng có sẵn trong SGK/SBT.

Mục tiêu của repo:

1. dựng đúng semantic của bảng;
2. bảng chỉ rộng/cao **vừa đủ cho nội dung và vùng viết cần thiết**;
3. đường kẻ, màu và style chung lấy từ config của repo để toàn sách đồng đều;
4. mỗi bảng chỉ khai báo phần cấu trúc/kích thước riêng của nó;
5. chỉ override border/style cục bộ khi nguồn thật sự yêu cầu khác default.

Không dùng `tasks`, TikZ, `\hspace`, `\makebox`, `resizebox` hoặc spacing tay để giả một semantic table.

## 1. Nguồn kỹ thuật chuẩn

Khi cần xác minh API/semantics của `tabularray`, ưu tiên:

- manual chính thức: https://mirrors.ctan.org/macros/latex/contrib/tabularray/tabularray.pdf
- CTAN package page: https://ctan.org/pkg/tabularray
- upstream source: https://github.com/lvjr/tabularray

TeX.SE/blog/forum chỉ dùng để tham khảo khi debug; không biến một mẹo cộng đồng thành convention của repo nếu manual/upstream không xác nhận.

## 2. Mental model bắt buộc

Agent phải tách bốn lớp:

- `colspec` / `rowspec`: **cấu trúc cột-hàng**;
- `cell`, `row`, `column`, `cells`, `rows`, `columns`: **style/kích thước dữ liệu**;
- `hline`, `vline`, `hlines`, `vlines`: **đường kẻ**;
- `tblr` / `longtblr`: **vòng đời của bảng** (một khối hay qua trang).

Không dùng một lớp để chữa lỗi của lớp khác.

Ví dụ: bảng quá rộng → sửa `colspec`/width/padding; không vội thu font. Border sai → sửa rule layer; không vẽ TikZ lên trên.

## 3. Nhận ra khi nào phải dùng bảng

Dùng `tblr`/`longtblr` khi ít nhất một điều đúng:

- mỗi hàng là một record có cùng trường dữ liệu;
- mỗi cột có ý nghĩa ổn định xuyên nhiều hàng;
- người đọc cần đối chiếu theo hàng và cột;
- vị trí một ô `(hàng, cột)` mang ý nghĩa;
- nguồn gốc đã trình bày dưới dạng bảng và cấu trúc đó có ý nghĩa.

Không dùng table khi:

- vài lựa chọn chỉ đứng cạnh nhau → dùng `tasks`;
- dữ liệu là list/prose dài → dùng list;
- lưới là đối tượng hình học, đường đi, ô tô màu, đếm hình → dùng TikZ/hình nguồn.

## 4. Fidelity của bảng nguồn

Phải giữ đúng:

- số hàng/cột có nghĩa;
- thứ tự hàng/cột;
- header;
- nội dung từng ô;
- ô trống có chủ đích;
- rowspan/colspan;
- đơn vị/ký hiệu/dấu câu;
- các đường phân vùng đặc biệt nếu chúng thể hiện cấu trúc.

Không rút gọn nội dung, gộp cột, đổi ô trống thành ký hiệu khác hoặc thêm emphasis chỉ để bảng dễ layout hơn.

## 5. `tblr` hay `longtblr`

### `tblr`

Dùng khi bảng hợp lý để nằm trong một khối/trang.

### `longtblr`

Dùng khi bảng dài có thể qua trang. Nếu header cần lặp lại, dùng `rowhead=<n>` thay vì copy thủ công sang nhiều bảng.

Không tự cắt một semantic table thành nhiều `tblr` chỉ để né page break nếu `longtblr` giải quyết đúng hơn.

## 6. Kích thước: mặc định vừa đủ, không ép full-width

Đây là convention quan trọng của repo.

### Bảng ngắn

Nếu dữ liệu ngắn và natural width đã hợp lý, **không đặt `width=\linewidth`**.

```tex
\begin{center}
\begin{tblr}{
  colspec={Q[l] *{4}{Q[c]}}
}
  $x$    & $-2$ & $-1$ & $0$ & $1$ \\
  $2x+1$ &      &      &     &     \\
\end{tblr}
\end{center}
```

### Bảng thực sự cần co giãn

Chỉ dùng `width=...`/`X` khi nội dung hoặc cấu trúc cần phân phối chiều rộng.

```tex
\noindent
\begin{tblr}{
  width=\linewidth,
  colspec={Q[c] X[2,l] X[3,l]}
}
  STT & Đại lượng & Nhận xét \\
  1   & ...       & ...      \\
\end{tblr}
```

Không ép bảng nhỏ thành full-width chỉ để nhìn "đều".

## 7. Chọn cột theo dữ liệu

### `Q`

Dùng cho dữ liệu ngắn/natural hoặc fixed width:

```tex
Q[l]
Q[c]
Q[r]
Q[c,m]
Q[c,m,2.2cm]
```

Phù hợp với số thứ tự, số ngắn, ký hiệu, đáp án Đ/S, nhãn ngắn.

### `X`

Dùng khi nội dung cần wrap và chia phần width còn lại:

```tex
X[l]
X[2,l]
X[3,c]
```

Không dùng toàn cột `Q[c]`/`c` cho prose dài rồi chữa overflow bằng font nhỏ.

## 8. Style chung của repo là source of truth

Repo hiện cấu hình `tblr`/`longtblr` với border mặc định:

```tex
\SetTblrInner[tblr, longtblr]{
    hlines = {solid, fg=base800},
    vlines = {solid, fg=base700}
}
```

Điều này có nghĩa:

- bảng grid thông thường **đã có hline/vline và màu chuẩn**;
- từng bảng không được lặp lại `hlines`, `vlines`, `fg=...` chỉ để "chắc chắn có viền";
- không tự chọn màu line khác cho từng bảng nếu nguồn không yêu cầu;
- nếu muốn đổi style chung cho toàn sách, thay đổi phải ở config sau approval, không copy override vào hàng loạt content.

Bảng thông thường nên chỉ còn cấu trúc riêng:

```tex
\begin{tblr}{
  colspec={Q[c] Q[c] Q[c]},
  rows={1cm}
}
  ...
\end{tblr}
```

## 9. Rule semantics: không vẽ lại đường đã có

Theo manual/upstream:

- `|` trong `colspec` tạo vertical rule;
- `|` trong `rowspec` tạo horizontal rule;
- `hlines`/`vlines` tạo rules hàng loạt;
- `hline{i}`/`vline{j}` nhắm rule/segment cụ thể;
- indexed rules cho phép nhiều đường trên cùng boundary.

Vì `|` là **rule thật**, không dùng nó như ký hiệu trang trí vô hại.

Sai với grid thông thường của repo:

```tex
\begin{tblr}{
  colspec={|Q[c]|Q[c]|Q[c]|},
  hlines,
  vlines
}
```

Đúng:

```tex
\begin{tblr}{
  colspec={Q[c] Q[c] Q[c]}
}
```

Nếu nguồn thật sự có double/multiple rule hoặc một boundary đặc biệt, dùng API `hline{...}` / `vline{...}` / rule index có chủ đích và audit PDF thật.

Không dùng `wd=0pt` như một convention "ẩn border" của repo; manual chỉ document `wd` là độ dày rule.

## 10. Alignment là semantic

Không căn giữa toàn bảng theo thói quen.

- prose → thường `l`;
- số/biểu thức ngắn → thường `c`;
- header ngắn → có thể `c`;
- nhãn hàng → thường `l`;
- nội dung nhiều dòng → chọn vertical alignment theo cách đọc (`m`, top, ...).

Có thể style theo layer:

```tex
\begin{tblr}{
  colspec={Q[l] X[l] Q[c]},
  cells={valign=m},
  row{1}={font=\bfseries, halign=c}
}
```

## 11. Math mode chỉ dùng khi cả vùng thật sự là toán

Có thể dùng `mode=math` ở cell/row/column nếu toàn vùng chỉ chứa toán.

```tex
\begin{tblr}{
  colspec={Q[l] Q[c]},
  column{2}={mode=math}
}
  Giá trị & -12 \\
  Kết quả & 3^2+4 \\
\end{tblr}
```

Nếu cột có prose tiếng Việt, giữ text mode và chỉ bọc phần toán bằng `$...$`.

Không bật math mode rộng đến mức chữ Việt chạy vào math font.

## 12. Span dùng API của `tabularray`

Dùng `\SetCell`/cell span:

```tex
\SetCell[c=2]{c} Tiêu đề hai cột & \\
```

```tex
\SetCell[r=2]{m} Nhãn hai hàng & ... \\
& ... \\
```

Không giả span bằng:

- `\makebox`;
- `\hspace` xuyên cell;
- width âm;
- TikZ overlay;
- xóa separator thủ công.

Sau span phải kiểm lại border và width ở PDF thật.

## 13. Padding/khoảng cách chỉnh đúng layer

Các key thường dùng:

- `colsep`, `leftsep`, `rightsep`;
- `rowsep`, `abovesep`, `belowsep`;
- `cells`, `rows`, `columns`;
- `cell{i}{j}`, `row{i}`, `column{j}`;
- `ht`, `wd` khi thật sự cần kích thước cụ thể.

Nếu bảng overfull, xử lý theo thứ tự:

1. kiểm tra bảng có bị ép full-width/natural-width sai không;
2. kiểm tra prose có đang nằm trong cột natural/fixed quá hẹp không;
3. đổi cột cần wrap sang `X`;
4. chỉnh hệ số `X`;
5. chỉnh `colsep` hợp lý;
6. kiểm tra span;
7. chỉ sau cùng mới cân nhắc font nhỏ hơn nếu style nguồn thật sự yêu cầu.

Không dùng `\resizebox` làm chiến lược mặc định.

## 14. Vùng trả lời trong bảng

Nếu học sinh điền trực tiếp vào cell:

- cell trống có border có thể chính là vùng viết;
- tăng chiều cao hàng/padding vừa đủ để viết;
- giữ chiều rộng phù hợp loại đáp án;
- checkbox/ký hiệu ngắn có thể dùng primitive trả lời sẵn có của repo nếu đúng semantic.

Không làm bảng thấp/rộng bất hợp lý chỉ để tiết kiệm trang.

## 15. Anti-patterns bắt buộc tránh

Không:

- giả bảng bằng `tasks`, TikZ hoặc spacing;
- ép mọi bảng thành `width=\linewidth`;
- lặp lại `hlines`/`vlines`/màu đã có trong config;
- dùng `|` trong `colspec`/`rowspec` cho grid bình thường khi config đã tạo rules;
- tự chọn border color riêng cho từng bảng;
- thu nhỏ toàn bảng bằng `resizebox` để che lỗi colspec;
- xuống dòng prose thủ công khi `X` có thể wrap;
- chia một semantic table thành nhiều bảng chỉ để né page break.

## 16. Checklist trước khi giao

### Semantic

- [ ] Đây thật sự là dữ liệu hàng-cột.
- [ ] Không dùng bảng chỉ để căn layout hai khối không liên quan.
- [ ] Không nhầm lưới hình học với table dữ liệu.

### Fidelity

- [ ] Hàng/cột/header/nội dung/span khớp nguồn.
- [ ] Không rút gọn nội dung để cứu layout.
- [ ] Border đặc biệt của nguồn được giữ nếu mang ý nghĩa.

### Size/layout

- [ ] Bảng dùng natural width nếu đã đủ; chỉ full-width khi thật sự cần.
- [ ] Cột prose dùng `X` khi cần wrap.
- [ ] Cột ngắn dùng `Q`/fixed hợp lý.
- [ ] Alignment đúng loại dữ liệu.
- [ ] Vùng trả lời đủ chỗ viết.

### Style/border

- [ ] Grid thông thường kế thừa `hlines`/`vlines` và màu từ config repo.
- [ ] Không lặp `hlines`, `vlines`, `fg` hoặc `|` chỉ để vẽ lại border mặc định.
- [ ] Override border chỉ tồn tại vì source thật sự khác default.
- [ ] Double/multiple rule chỉ xuất hiện có chủ đích.

### Build

- [ ] Không có overfull/overlap do bảng thuộc scope thay đổi.
- [ ] Không có missing glyph do math mode sai.
- [ ] Audit PDF thật: width hợp lý, chữ đọc được, border đồng đều, cell không đè nhau.

## 17. Nguyên tắc chốt cho Agent

> **Mỗi bảng chỉ khai báo cái riêng của bảng đó. Cái chung của cả sách phải lấy từ config chung.**

Trong repo này, border và màu grid đã là policy chung. Vì vậy Agent tập trung vào semantic, `colspec`/`rowspec`, span, alignment, kích thước và vùng viết; không vẽ lại line/color ở từng bảng trừ khi bảng nguồn thật sự là ngoại lệ.

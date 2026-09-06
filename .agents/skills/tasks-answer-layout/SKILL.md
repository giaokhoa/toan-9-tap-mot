---
name: tasks-answer-layout
description: "Trình bày lựa chọn/câu ngắn bằng package tasks và thiết kế vùng trả lời linh động theo độ dài thực tế: chọn số cột có chủ đích, cho item dài span nhiều cột, dùng answerbox cố định cho ký hiệu ngắn và answerline cho nội dung dài hơn và tránh layout cứng phải sửa tay."
metadata:
  language: "vi"
  version: "1.0.0"
  scope: "latex,tasks,multiple-choice,columns,answerbox,answerline,worksheet,layout"
---

# `tasks` và vùng trả lời linh động

Dùng skill này khi nội dung có:

- các lựa chọn A/B/C/D hoặc nhiều ý ngắn nên xếp ngang;
- danh sách các biểu thức/câu con có độ dài khác nhau;
- bảng/chuỗi cần ô điền đáp án;
- chỗ trống inline hoặc vùng trả lời mà kích thước phải tùy theo dạng câu;
- yêu cầu giảm sửa tay do layout cứng.

Mục tiêu là để Agent **tự quyết định layout theo nội dung**, không mặc định “4 lựa chọn = 4 cột” và không mặc định “mọi chỗ trống = ô vuông 14pt”.

Skill này không cho phép sửa/rút gọn câu nguồn để làm vừa cột. Nếu nội dung không vừa, đổi layout.

## 1. Hiểu đúng package `tasks`

`tasks` được thiết kế cho các danh sách nhiều cột **đếm theo hàng ngang**, rất hợp kiểu bài tập toán. Nhưng nó không phải `enumerate` được chia cột.

Các giới hạn quan trọng từ manual:

- item được tách bằng `\task`, không phải `\item`;
- environment là pseudo-environment: body được đọc trước rồi mới xử lý;
- **không nest `tasks` trong `tasks`**;
- không dùng verbatim trực tiếp trong item;
- không page-break bên trong một item; chỉ có thể ngắt giữa các item;
- số cột là số cột bằng nhau do `\begin{tasks}(n)` chỉ định.

Vì vậy, nếu một item dài nhiều dòng và có nguy cơ cần ngắt trang, **không ép vào `tasks`**; dùng `questions`, `enumerate`, `itemize` hoặc layout khác phù hợp hơn.

## 2. Quy tắc chọn `tasks` hay list thường

Dùng `tasks` khi các item:

- cùng cấp logic;
- tương đối ngắn;
- có lợi khi quét mắt theo hàng ngang;
- không chứa cấu trúc list con phức tạp;
- không cần page-break bên trong item.

Không dùng `tasks` chỉ vì có nhiều ý. Với câu dài, chứng minh, lời văn nhiều dòng hoặc mỗi item có sub-list, dùng list thường.

## 3. Không suy số cột từ số lượng item

**Sai:** có 4 lựa chọn thì luôn `(4)`.

**Đúng:** chọn số cột theo **độ rộng tự nhiên của item dài nhất và độ cân bằng của cả hàng**.

Heuristic ban đầu:

- `(4)`: ký hiệu, số, biểu thức rất ngắn, lựa chọn 1–3 từ;
- `(3)`: cụm ngắn, biểu thức vừa;
- `(2)`: câu/biểu thức dài hơn, mỗi item có thể wrap nhẹ;
- `(1)` hoặc list thường: item dài, nhiều mệnh đề.

Đây chỉ là điểm khởi đầu. Sau build, nhìn PDF thật và điều chỉnh. **Không rút gọn chữ nguồn** để giữ số cột đã chọn.

## 4. Item dài phải span cột thay vì phá layout

Upstream `tasks` có ba cơ chế quan trọng:

### 4.1. `\task*`: dùng phần cột còn lại của hàng

```tex
\begin{tasks}(4)
  \task $2+3$
  \task $5^2$
  \task* Một lựa chọn dài cần dùng phần chiều rộng còn lại của hàng.
\end{tasks}
```

Dùng khi một item dài hơn các item trước và hợp lý để ăn phần còn lại của chính hàng đó.

### 4.2. `\task*(n)`: span tối đa `n` cột

```tex
\begin{tasks}(4)
  \task $A$
  \task $B$
  \task*(2) Nội dung này cần khoảng hai cột.
\end{tasks}
```

Đây là công cụ chính cho danh sách **mixed-width**. Nếu hàng chỉ còn ít cột hơn `n`, package dùng tối đa số cột còn lại.

### 4.3. `\task!`: chiếm cả hàng

```tex
\begin{tasks}(4)
  \task $A$
  \task $B$
  \task! Lựa chọn/câu này phải dùng toàn bộ chiều rộng.
  \task $C$
  \task $D$
\end{tasks}
```

Dùng khi item dài rõ ràng không nên bị bó vào cột nhỏ.

### 4.4. `\startnewitemline` chỉ là phương án cuối

Manual cảnh báo lệnh này chỉ ép sang hàng mới nhưng **không đổi width của item**; muốn tận dụng full width thường phải dùng trick như `\rlap`, dễ chọc ra lề.

Vì vậy convention của repo:

1. ưu tiên chọn số cột đúng;
2. sau đó dùng `\task*`, `\task*(n)`, `\task!`;
3. chỉ dùng `\startnewitemline` khi đã hiểu rõ box width và đã build kiểm tra.

## 5. Local options quan trọng hơn global setting

`config/packages/tasks.tex` chỉ là baseline. Mỗi danh sách được phép override theo content:

```tex
\begin{tasks}[
  label=\Alph*.,
  label-width=1.4em,
  label-offset=.4em,
  item-indent=1.8em,
  column-sep=1em,
  after-item-skip=4pt
](2)
  \task ...
  \task ...
\end{tasks}
```

Các key cần biết:

- `label`: nội dung nhãn;
- `label-format`: format nhãn, nên dùng thay vì nhét style vào `label` khi có thể;
- `ref`: representation khi `\ref`;
- `label-width`: box dành cho nhãn;
- `label-offset`: khoảng giữa nhãn và item;
- `item-indent`: tổng vùng horizontal trước text item;
- `column-sep`: khoảng giữa các cột;
- `label-align`: left/right/center;
- `before-skip`, `after-skip`, `after-item-skip`;
- `start`, `resume`, `counter` khi thực sự cần continuation;
- `debug`: vẽ khung box để debug layout.

Theo manual, nếu muốn text item align với textblock, thường cần suy nghĩ theo quan hệ:

`item-indent ≈ label-width + label-offset`.

Không để `item-indent` lớn một cách máy móc rồi bù bằng spacing tay.

## 6. Dùng `debug` khi layout khó

Khi item lệch, không rõ vì sao wrap hoặc khoảng trống kỳ lạ:

```tex
\begin{tasks}[debug](3)
  ...
\end{tasks}
```

`debug` cho thấy box nhãn và box item. Dùng nó để hiểu width, sau đó **bỏ `debug` trước commit cuối**.

Không sửa bằng chuỗi `\hspace`, `\phantom`, `\rlap` ngẫu nhiên trước khi biết box nào đang gây vấn đề.

## 7. Quy tắc cho trắc nghiệm/lựa chọn

### 7.1. Bảo toàn nguyên văn lựa chọn

Không viết lại lựa chọn cho ngắn để nhét 4 cột.

Nếu A/B/C ngắn nhưng D dài:

- dùng `(2)` cho toàn bộ nếu cân đối hơn; hoặc
- dùng `(4)` và cho D `\task!`/`\task*(n)` nếu đúng nhịp thị giác.

### 7.2. Không ép các item phải cùng số dòng

`tasks` cho phép item wrap. Mục tiêu là đọc dễ, không phải tạo lưới hình học tuyệt đối. Nếu một hàng có chênh quá lớn, đổi span hoặc số cột.

### 7.3. Custom label

Một item có thể override nhãn bằng optional argument của `\task`. Chỉ dùng nếu **nguồn thực sự có nhãn khác**; không dùng để hard-code numbering bừa.

## 8. `\answerbox` là primitive cố định

`\answerbox` của repo giữ đúng thiết kế ban đầu: một ô vuông `14pt × 14pt`, baseline `3pt`, có khoảng hở ngang `4pt` mỗi bên.

```tex
\answerbox
```

Không truyền optional argument để thay width, height hoặc padding cho `\answerbox`. Nếu nội dung cần nhiều chỗ viết hơn, chọn primitive semantic khác (`\answerline*`, `\answerline`) hoặc chỉnh layout của container (cell/row/column), không biến `\answerbox` thành hộp co giãn.

## 9. Chọn vùng trả lời theo loại câu

Agent **không cần viết lời giải/đáp án ẩn**. Chỉ chọn primitive theo đúng loại nội dung học sinh cần điền.

### Dùng `\answerbox`

- một ký hiệu: `∈`, `∉`, `|`, `∤`, `<`, `>`, `=`;
- Đ/S;
- một chữ cái/lựa chọn;
- checkbox hoặc một chữ số rất ngắn.

### Dùng `\answerline*`

- số có nhiều chữ số;
- từ/cụm từ ngắn;
- biểu thức có độ dài khó đoán;
- chỗ trống inline nhưng không có lý do phải đóng khung.

```tex
Số cần điền là \answerline*[3cm].
```

### Dùng `\answerline`

- câu trả lời độc lập;
- phép tính cần trình bày;
- giải thích ngắn.

Không ghép nhiều `\answerbox` để giả một dòng trả lời, trừ khi nguồn thật sự có các ô tách rời.

## 10. Ô trong bảng: để container quyết định kích thước

Trong `tabular`, `tblr` và bảng nhiều cột:

- nếu cell có border và bản thân cell là chỗ điền, để cell trống; không đặt một hộp kích thước tùy ý bên trong;
- checkbox/ký hiệu đơn dùng `\answerbox` cố định;
- nếu nguồn có dòng trống trong cell, dùng `\answerline*`;
- chiều rộng cột, chiều cao hàng và padding phải chỉnh bằng API của table (`colspec`, `wd`, `ht`, row/column/cell styles), không bằng tham số của `\answerbox`.

## 11. Phối hợp `tasks` và vùng trả lời

Không đặt answer primitive trước rồi mới ép layout quanh nó. Quy trình đúng:

1. xác định nội dung item nguyên văn;
2. ước lượng width tự nhiên của item **kèm vùng trả lời**;
3. chọn số cột;
4. quyết định item nào cần `\task*(n)`/`\task!`;
5. chọn `answerbox`/`answerline*` phù hợp;
6. build và điều chỉnh local option.

Ví dụ:

```tex
\begin{tasks}[column-sep=1em](3)
  \task $24\ \answerbox\ 6$
  \task $35\ \answerbox\ 5$
  \task* $125 = \answerline*[2.5em] \cdot 5$;
  \task! Viết một câu trả lời ngắn: \answerline*[5cm]
\end{tasks}
```

Điểm quan trọng là item dài **tự được cho nhiều cột**, không bắt text wrap trong một cột hẹp vì layout đã chọn cứng từ đầu.

## 12. Anti-patterns phải tránh

### 12.1. Bốn lựa chọn dài trong bốn cột

```tex
\begin{tasks}(4)
  \task Một câu rất dài ...
  \task Một câu rất dài ...
  \task Một câu rất dài ...
  \task Một câu rất dài ...
\end{tasks}
```

Nếu phải dùng font nhỏ hoặc sửa chữ để vừa, layout sai.

### 12.2. Dùng `\hspace` để “cân” item

Không căn từng lựa chọn bằng tay. Chỉnh `label-width`, `label-offset`, `item-indent`, `column-sep`, span.

### 12.3. Mọi blank đều là `\answerbox`

Một cụm từ 5 chữ không nên có ô 14pt; một ký hiệu đơn không cần một `\answerline` dài cả trang.

### 12.4. Chain ô để giả một line

```tex
\answerbox\answerbox\answerbox\answerbox\answerbox
```

chỉ hợp lệ nếu nguồn có năm ô riêng. Nếu không, dùng một rectangle hoặc line.

## 13. Checklist trước khi giao

### `tasks`

- [ ] `tasks` thực sự phù hợp; item không cần nested tasks/pagebreak bên trong.
- [ ] Số cột được chọn theo độ dài item, không theo số lượng item.
- [ ] Item dài dùng `\task*`, `\task*(n)` hoặc `\task!` khi hợp lý.
- [ ] Không dùng `\startnewitemline`/`\rlap` nếu chưa thật sự cần.
- [ ] `label-width`, `label-offset`, `item-indent`, `column-sep` cân đối.
- [ ] Không có `debug` sót trong bản cuối.

### Vùng trả lời

- [ ] Kích thước box phù hợp lượng ký tự dự kiến.
- [ ] Ô vuông chỉ dùng cho đáp án rất ngắn.
- [ ] Rectangle/line được dùng khi câu trả lời dài hơn.
- [ ] Padding của box không làm bảng/list overfull.
- [ ] Không sinh đáp án ẩn chỉ để tính kích thước.

### Fidelity/build

- [ ] Không sửa/rút gọn câu nguồn để cứu layout.
- [ ] Không thêm nhãn trả lời AI tự nghĩ.
- [ ] Không có `Overfull \hbox/\vbox` hoặc warning layout liên quan ở final log.
- [ ] PDF thật nhìn cân, không chỉ source code “trông gọn”.

## 14. Nguồn kỹ thuật đã dùng để xây skill

- CTAN package page: https://ctan.org/pkg/tasks
- Upstream repository: https://github.com/cgnieder/tasks
- Upstream manual source: https://github.com/cgnieder/tasks/blob/master/tasks-manual.tex

Các capability được skill sử dụng trực tiếp từ manual gồm:

- `\begin{tasks}(n)`;
- `\task*`, `\task*(n)`, `\task!`;
- `\startnewitemline` và cảnh báo về width;
- `label`, `label-format`, `ref`;
- `label-width`, `label-offset`, `item-indent`, `column-sep`, `label-align`;
- `before-skip`, `after-skip`, `after-item-skip`;
- `start`, `resume`, `counter`, `debug`;
- các giới hạn pseudo-environment, không nest và không pagebreak trong item.
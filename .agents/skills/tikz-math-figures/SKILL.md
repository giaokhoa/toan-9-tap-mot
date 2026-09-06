---
name: tikz-math-figures
description: "Phân loại hình trước khi redraw và chỉ dùng TikZ cho hình toán học có thể mô hình hóa chính xác: mọi điểm quan trọng có tên, nhãn dễ đọc, quan hệ dựng bằng calc/intersections/angles thay vì tọa độ xấp xỉ, đồng thời giữ tuyệt đối dữ kiện của nguồn."
metadata:
  language: "vi"
  version: "1.1.0"
  scope: "latex,tikz,pgf,geometry,math-figures,nodes,coordinates,labels,intersections,angles,fidelity,image-triage"
---

# TikZ cho hình toán học có ngữ nghĩa

Dùng skill này khi issue cần **vẽ mới, dựng lại, sửa hoặc audit** hình toán học bằng TikZ/PGF, đặc biệt hình học phẳng, hình có điểm được đặt tên, đường phụ, giao điểm, góc, trung điểm, đường tròn hoặc sơ đồ mà vị trí các đối tượng mang ý nghĩa toán học.

Mục tiêu không phải là “vẽ cho nhìn giống”. Mục tiêu là tạo **một mô hình hình học đúng, đọc được và bảo trì được**, rồi mới render mô hình đó thành hình.

Skill này không thay thế quy tắc fidelity của repository. Nếu hình đến từ SGK/SBT, mọi chữ, nhãn, số, ký hiệu, dữ kiện và quan hệ trong nguồn vẫn phải giữ nguyên.

## 0. Decision gate: không phải hình nào cũng nên redraw bằng TikZ

**Trước khi viết bất kỳ `\draw` nào**, quyết định hình thuộc nhóm nào.

### 0.1. TikZ-able

Chỉ redraw bằng TikZ khi hình có cấu trúc rõ ràng và có thể mô hình hóa/kiểm chứng chính xác, ví dụ:

- điểm, đoạn thẳng, tia, đường thẳng, đường tròn;
- tam giác, tứ giác, đa giác, trục số;
- hình có quan hệ trung điểm, giao điểm, song song, vuông góc, đối xứng;
- sơ đồ cây, sơ đồ phân tích, lưới toán học đơn giản;
- sơ đồ/ký hiệu đơn giản mà mọi nhãn, số và quan hệ đều đọc được chắc chắn từ nguồn.

Nếu chọn TikZ, agent phải có khả năng trả lời: **những object nào tồn tại, quan hệ nào định nghĩa chúng, dữ kiện nào phải giữ nguyên**.

### 0.2. Không TikZ-able

Không cố redraw bằng TikZ khi hình là:

- ảnh thực tế, người, động vật, đồ vật/cảnh vật có chi tiết tự nhiên;
- tranh minh họa;
- logo, biểu tượng phức tạp, hình nghệ thuật;
- texture, phối cảnh, ảnh cắt-ghép nhiều chi tiết;
- hình có chữ/chi tiết không đọc chắc chắn;
- hình mà agent phải đoán nét, màu, số lượng object hoặc vị trí;
- bất kỳ hình nào mà kết quả chỉ có thể “trông gần giống”.

Trong trường hợp này:

1. ưu tiên asset gốc;
2. nếu cần thì crop sạch đúng vùng nguồn;
3. nếu chưa có asset/crop thì **không bịa hình thay thế**.

Đặt comment source-only ngay vị trí cần hình:

```tex
% TODO(HINH-NGUON): bo sung asset/crop dung tu nguon; khong redraw xap xi bang TikZ.
```

Có thể thêm tên nguồn/trang để giáo viên tìm nhanh. Comment này **không được render thành placeholder nhìn thấy trên bản học sinh**.

Nếu đang redraw mà phát hiện không thể bảo đảm fidelity 100%, dừng và quay lại asset/crop/TODO; không “cố hoàn thiện” một bản xấp xỉ.

## 1. Nguyên tắc cốt lõi: dựng hình học trước, vẽ sau

Trước khi viết `\draw`, lập inventory nội bộ:

- các **điểm toán học có tên**: $A,B,C,M,O,\ldots$;
- các điểm dựng phụ/giao điểm/trung điểm/chân đường vuông góc;
- đoạn thẳng, tia, đường thẳng, đường tròn, cung;
- quan hệ: thẳng hàng, giao nhau, trung điểm, vuông góc, song song, bằng nhau;
- nhãn cạnh, nhãn góc, số đo, đơn vị;
- lớp nào là đường chính, đường phụ, vùng tô, chú thích.

Sau đó mới chọn tọa độ ban đầu tối thiểu. Những điểm có thể **suy ra từ quan hệ hình học** phải được suy ra bằng TikZ (`calc`, `intersections`, `angles`, ...), không gõ tọa độ xấp xỉ chỉ để hình trông đúng.

### Bắt buộc

- Mỗi điểm toán học quan trọng phải có **tên TikZ ổn định** như `(A)`, `(B)`, `(M)`, `(O)`.
- Không lặp lại cùng một tọa độ literal ở nhiều chỗ để giả vờ đó là cùng một điểm.
- Cạnh/đường phải tham chiếu điểm đã đặt tên: `\draw (A)--(B);`, không vẽ lại bằng hai cặp số độc lập.
- Điểm suy ra phải có tên và được dùng lại ở mọi path/nhãn liên quan.

## 2. Điểm và node: code phải nói được “đây là điểm A”

### 2.1. Điểm có marker trong nguồn

Nếu nguồn thể hiện điểm bằng chấm/marker, dùng **named node** làm chính điểm đó, ví dụ:

```tex
\tikzset{
  geom point/.style={circle, fill=base950, inner sep=1.2pt, outer sep=0pt},
  geom label/.style={font=\small, inner sep=1pt}
}

\node[geom point, label={[geom label]above left:$A$}] (A) at (0,0) {};
\node[geom point, label={[geom label]above right:$B$}] (B) at (4,0) {};
\draw (A)--(B);
```

Node `(A)` vừa là điểm hình học để path tham chiếu, vừa là marker nhìn thấy.

### 2.2. Nguồn không có marker

**Không tự thêm chấm** chỉ vì code tiện hơn. Dùng named coordinate làm điểm hình học, rồi tạo label riêng:

```tex
\coordinate (A) at (0,0);
\coordinate (B) at (4,0);
\draw (A)--(B);
\node[above left=1pt of A, inner sep=1pt] {$A$};
\node[above right=1pt of B, inner sep=1pt] {$B$};
```

Tức là: điểm vẫn có tên semantic trong code, nhưng output không thêm dữ kiện thị giác mà nguồn không có.

### 2.3. Điểm dựng phụ

Điểm dựng phụ vẫn phải có tên nếu được dùng từ hai lần trở lên hoặc tham gia một quan hệ hình học. Có thể không có label nhìn thấy nếu nguồn không label nó.

```tex
\coordinate (M) at ($(A)!0.5!(B)$);
\draw[dashed] (C)--(M);
```

Không viết hai lần `($(A)!0.5!(B)$)` nếu về mặt toán học đó là cùng một điểm $M$.

## 3. Nhãn điểm phải dễ đọc

Nhãn là **node chữ**, không phải phần trang trí phụ. Audit nhãn ở kích thước PDF thật, không chỉ ở zoom lớn.

### Quy tắc

- Dùng `label=...`, anchor (`above`, `below right`, ...) hoặc node riêng có offset rõ ràng.
- Chọn phía đặt nhãn để **không đè lên cạnh, cung, marker hoặc nhãn khác**.
- Dùng `label distance`/offset nhỏ có chủ đích thay vì chèn khoảng trắng bằng tay trong text.
- Khi nhiều nhãn gần nhau, ưu tiên node riêng để kiểm soát anchor/xshift/yshift chính xác.
- Không dùng font cực nhỏ để cứu layout. Mặc định giữ cỡ chữ tài liệu hoặc `\small`; không xuống dưới `\footnotesize` nếu không có lý do từ nguồn/reviewer.
- Không dùng `transform shape` chỉ để thu nhỏ toàn bộ label cùng hình.

PGF/TikZ mặc định **không áp dụng scale/rotation ngoài lên node text**, vì thông thường chữ không nên bị scale theo graphic. Hãy tận dụng hành vi này để geometry co giãn mà nhãn vẫn đọc được.

### Anti-pattern

```tex
\begin{tikzpicture}[scale=.55, transform shape]
  ...
\end{tikzpicture}
```

nếu mục đích chỉ là ép hình vào trang. Cách này làm cả chữ điểm nhỏ theo. Hãy sửa mô hình/khung hình hoặc style chung trước.

## 4. Dùng đúng library theo quan hệ hình học

Repository nên load sẵn bộ tối thiểu:

```tex
\usetikzlibrary{calc, intersections, backgrounds, positioning, angles, quotes}
```

### 4.1. `calc`: điểm nội suy, trung điểm, phép chiếu

Trung điểm:

```tex
\coordinate (M) at ($(A)!0.5!(B)$);
```

Một điểm chia đoạn theo tỉ lệ:

```tex
\coordinate (P) at ($(A)!0.35!(B)$);
```

Ưu tiên biểu thức quan hệ với `(A)`, `(B)` hơn tọa độ số mới.

### 4.2. `intersections`: giao điểm thật, không căn mắt

```tex
\path[name path=AC] (A)--(C);
\path[name path=BD] (B)--(D);
\path[name intersections={of=AC and BD, by=O}];
```

Từ đó dùng `(O)` cho marker, label, đoạn thẳng hoặc công thức tiếp theo.

Không ước lượng `O` bằng `(1.97,1.42)` nếu $O$ được định nghĩa là giao điểm của hai đường.

Nếu có nhiều giao điểm, dùng `by={...}` hoặc `sort by=<path>` để đặt tên có chủ đích; không dựa mù vào `intersection-1` khi thứ tự có thể khó đọc/bảo trì.

### 4.3. `angles` + `quotes`: góc phải dựa vào điểm có tên

```tex
\pic[draw, angle radius=5mm, "$\alpha$"] {angle=A--B--C};
\pic[draw, angle radius=4mm] {right angle=A--B--C};
```

Library `angles` yêu cầu $A,B,C$ là **node/coordinate đã có tên**. Đây cũng là convention bắt buộc của skill này.

Chỉ vẽ marker góc vuông, cung góc hoặc số đo khi nguồn có hoặc issue yêu cầu; không tự thêm để “giải thích cho đẹp”.

### 4.4. `positioning`: đặt label/chú thích theo quan hệ, không theo số tuyệt đối rải rác

```tex
\node[above right=2pt of A] {$A$};
```

Dùng anchor/positioning để ý định code rõ hơn `\node at (0.17,0.23)`.

## 5. Tọa độ ban đầu phải tối thiểu và có ý nghĩa

Không phải mọi tọa độ số đều xấu. Với một tam giác, có thể cần chọn ba điểm cơ sở để tạo hình cân đối. Nhưng sau khi chọn các điểm độc lập đó, các điểm phụ phải suy ra từ chúng.

Ví dụ tốt:

```tex
\coordinate (A) at (0,0);
\coordinate (B) at (5,0);
\coordinate (C) at (1.6,3.2);
\coordinate (M) at ($(A)!0.5!(B)$);
\draw (A)--(B)--(C)--cycle;
\draw[dashed] (C)--(M);
```

Ví dụ kém:

```tex
\draw (0,0)--(5,0)--(1.6,3.2)--cycle;
\draw[dashed] (1.6,3.2)--(2.5,0);
\node at (-.2,-.2) {$A$};
\node at (5.2,-.2) {$B$};
```

Dù output có thể giống nhau, bản thứ hai không biểu diễn quan hệ `M` là trung điểm và khó sửa/audit.

## 6. Style phải nói vai trò, không chỉ nói màu

Style dùng trong TikZ nên mang ý nghĩa semantic:

```tex
\tikzset{
  geom edge/.style={semithick},
  geom construction/.style={thin, dashed},
  geom point/.style={circle, fill=base950, inner sep=1.2pt, outer sep=0pt},
  geom label/.style={font=\small, inner sep=1pt}
}
```

Có thể override nét/kích thước khi nguồn bắt buộc. Không biến `redline`, `bluedot` thành abstraction chính nếu điều thực sự cần phân biệt là cạnh chính, đường dựng, điểm, vector, đường cắt, v.v.

### Không hard-code style lặp lại trong content

- Nếu cùng một TikZ style/font/line width/node padding được dùng ở nhiều hình/file, đưa nó về config hoặc macro/style dùng chung; không copy-paste literal khắp content.
- Content nên chứa **dữ kiện hình và quan hệ hình học**, không phải bộ design token riêng cho từng bài.
- Không dùng `\scriptsize`, `\fontsize`, `scale`, `xshift/yshift` hàng loạt chỉ để cứu layout nếu vấn đề thuộc style chung.
- Tọa độ/offset cục bộ vẫn hợp lệ khi chúng thể hiện cấu trúc riêng của đúng hình; điều bị cấm là vá style thẩm mỹ lặp lại nhiều nơi.

Nếu thấy một pattern layout hình lặp lại nhưng repo chưa có style chung, ghi nhận và bổ sung style/config chung trước khi nhân rộng.

## 7. Thứ tự render: đường trước, nhãn sau

Mặc định sắp xếp:

1. vùng nền/tô;
2. đường dựng phụ;
3. cạnh/đường chính;
4. marker điểm;
5. nhãn điểm;
6. nhãn cạnh/góc/chú thích.

Lý do: node label thường phải nằm trên path để chữ không bị đường kẻ xuyên qua. Với vùng nền, dùng `backgrounds`/background layer thay vì vẽ fill muộn rồi che mất điểm/chữ.

Không phụ thuộc ngẫu nhiên vào thứ tự câu lệnh dài hàng chục dòng; nếu hình phức tạp, chia scope/layer theo vai trò.

## 8. Các ký hiệu hình học phải bám vào đối tượng semantic

- Nhãn cạnh: đặt trên path bằng `node[midway,...]` hoặc quotes; không tính midpoint bằng mắt.
- Mũi tên: đặt trên đúng path/tia/vector đã tham chiếu node.
- Góc: dùng `pic {angle=...}`/`right angle=...` khi phù hợp.
- Giao điểm: dùng `intersections`.
- Trung điểm/chia đoạn: dùng `calc`.
- Đường tròn/tâm: giữ node tâm riêng; các điểm đặc biệt trên đường tròn phải được đặt tên nếu được dùng tiếp.

Nếu một ký hiệu như tick bằng nhau, mũi tên song song, cung góc hoặc đường vuông góc là **dữ kiện của bài**, không được bỏ vì “hình vẫn hiểu được”.

## 9. Khi asset/crop an toàn hơn TikZ

Dùng asset/crop nguồn nếu một trong các điều sau đúng:

- hình có texture/ảnh thực/phối cảnh phức tạp;
- có nhiều chi tiết nhỏ mà chưa xác minh được hết;
- chữ/nhãn trong hình không đọc chắc chắn;
- phải đoán vị trí, số lượng hoặc quan hệ;
- redraw chỉ nhằm “trông tương tự”, không thể chứng minh fidelity.

Nếu chưa có asset, dùng `% TODO(HINH-NGUON): ...` source-only thay vì redraw xấp xỉ.

TikZ phù hợp nhất khi cấu trúc toán học rõ ràng và có thể dựng từ quan hệ semantic.

## 10. Quy trình dựng một hình

1. Mở nguồn và inventory toàn bộ dữ kiện nhìn thấy.
2. **Phân loại TikZ-able / không TikZ-able.**
3. Nếu không TikZ-able: dùng asset/crop hoặc TODO source-only rồi dừng redraw.
4. Nếu TikZ-able: xác định điểm/object độc lập và object suy ra.
5. Đặt tên semantic cho mọi điểm quan trọng.
6. Chọn ít tọa độ cơ sở nhất có thể.
7. Dựng điểm phụ bằng `calc`/`intersections`.
8. Vẽ đường/nền.
9. Vẽ marker điểm đúng theo nguồn.
10. Đặt nhãn bằng anchor/label/positioning.
11. Thêm góc, tick, số đo, mũi tên và chú thích đúng nguồn.
12. Build ở kích thước trang thật.
13. Audit label collision, clipping và fidelity.

## 11. Checklist audit bắt buộc

### Triage

- [ ] Đã quyết định hình thực sự TikZ-able trước khi redraw.
- [ ] Ảnh/tranh/logo/phối cảnh phức tạp không bị redraw xấp xỉ.
- [ ] Nếu thiếu asset cho hình không TikZ-able, có `% TODO(HINH-NGUON): ...` đúng vị trí và không render ra PDF.

### Semantic code

- [ ] Mọi điểm toán học quan trọng có tên `(A)`, `(B)`, ... rõ ràng.
- [ ] Không có cùng một điểm được lặp bằng nhiều tọa độ literal độc lập.
- [ ] Trung điểm/giao điểm/điểm chia đoạn được suy ra từ điểm gốc khi có thể.
- [ ] Cạnh/đường tham chiếu node/coordinate đã đặt tên.
- [ ] Style phân biệt vai trò hình học, không chỉ màu.
- [ ] Style/font/spacing lặp lại nhiều nơi không bị hard-code trong content.

### Readability

- [ ] Tất cả nhãn điểm đọc rõ ở kích thước PDF thật.
- [ ] Không có nhãn đè cạnh, cung, marker hoặc nhãn khác.
- [ ] Không thu nhỏ text bằng `transform shape` chỉ để ép layout.
- [ ] Marker điểm không che label và không lớn quá mức.

### Fidelity

- [ ] Không thiếu nhãn, số, ký hiệu, tick, góc, mũi tên, chú thích hoặc object có trong nguồn.
- [ ] Không tự thêm marker/góc/chú thích mà nguồn không có, trừ khi issue yêu cầu.
- [ ] Quan hệ hình học đúng, không chỉ “nhìn gần giống”.
- [ ] Nếu không bảo đảm 100%, đã chuyển sang asset/crop/TODO nguồn.

### Build

- [ ] Hình không vượt lề/cắt mất nội dung.
- [ ] Build thành công.
- [ ] Nếu issue/reviewer yêu cầu warning audit, final log đã được đọc và warning do hình được phân loại/xử lý đúng scope; không tự biến mọi warning thành CI failure.

## 12. Nguồn kỹ thuật đã dùng để xây skill

Ưu tiên các nguồn chính thức sau khi cần tra chi tiết:

- PGF/TikZ Manual — Nodes and Edges: https://tikz.dev/tikz-shapes
- PGF/TikZ Manual — Specifying Coordinates / intersections: https://tikz.dev/tikz-coordinates
- PGF/TikZ Manual — Calc Library: https://tikz.dev/library-calc
- PGF/TikZ Manual — Angle Library: https://tikz.dev/library-angle
- PGF/TikZ Manual — Euclid tutorial (semantic construction with named coordinates/intersections): https://tikz.dev/tutorial-Euclid

Các nguyên tắc đặc biệt quan trọng từ manual:

- node có thể đặt tên và được tham chiếu lại bằng anchor;
- `intersections` tạo named coordinates từ giao của named paths;
- `angles` làm việc với các điểm đã có tên;
- transformation bên ngoài mặc định không scale node text, vì thông thường không nên scale chữ theo graphic;
- `label distance`, anchors và `positioning` cho phép tách vị trí nhãn khỏi tọa độ hình học của điểm.
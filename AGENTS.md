# Quy tắc biên soạn repository sách Toán

## Workflow GitHub bắt buộc

Mỗi Agent làm một issue phải làm việc qua **branch riêng + pull request**, không commit trực tiếp vào `main`.

- Trước khi sửa, cập nhật từ `main` hiện tại rồi tạo branch riêng cho issue.
- Mặc định một issue dùng một branch. Tên branch nên có số issue và mô tả ngắn, ví dụ `lesson/issue-12-so-nguyen` hoặc `fix/issue-12-numbering`.
- Không dùng lại branch của issue đã hoàn thành cho issue khác.
- Toàn bộ commit của issue phải nằm trên branch đó.
- Khi hoàn thành và build/check đạt yêu cầu, Agent phải mở PR từ branch của mình vào `main`; không tự đẩy thay đổi thẳng vào `main`.
- PR phải ghi rõ issue được xử lý và chứa closing keyword đúng dạng `Closes #<issue-number>` trong body. Không chỉ viết `#<issue-number>` hoặc `Related to #...`.
- Nếu một PR được yêu cầu xử lý nhiều issue, ghi một dòng `Closes #...` cho từng issue thực sự được PR hoàn thành.
- **Không đóng issue thủ công trước khi merge PR.** GitHub sẽ tự đóng issue khi PR có `Closes #...` được merge vào default branch.
- Nếu PR bị đóng mà **không merge**, issue phải tiếp tục mở; không tự đóng issue cho một thay đổi chưa vào `main`.
- Trước khi giao, Agent phải kiểm tra PR đang trỏ đúng base `main`, có `Closes #...`, và CI/check cần thiết đã chạy.
- **Build pass không đồng nghĩa với hoàn thành.** Sau build, Agent bắt buộc phải kiểm tra lại output thực tế của đúng thay đổi vừa làm trước khi merge. Với thay đổi layout/typography/TikZ/table/list/spacing, phải mở PDF artifact ở các trang/vùng bị tác động và đối chiếu trực quan; với thay đổi logic/numbering/content generation phải kiểm output render tương ứng; đồng thời đọc final LaTeX log cho lỗi/warning mới thuộc scope.
- Nếu thay đổi có mục tiêu trước/sau về hình thức, Agent phải so sánh artifact trước và sau hoặc ít nhất kiểm trực tiếp các case đại diện đã gây vấn đề. Không được kết luận chỉ từ code diff hay trạng thái CI xanh.
- Nếu không thể mở/kiểm artifact sau build, phải báo rõ verification còn thiếu và **không merge như thể đã xác minh**.
- CI hiện là **build gate**, không phải warning gate tuyệt đối. Không tự sửa workflow để biến mọi LaTeX warning thành lỗi CI. Khi issue/reviewer yêu cầu audit warning, Agent phải đọc final LaTeX pass, phân loại warning và sửa/báo đúng phạm vi; warning audit có thể được thực hiện riêng sau mỗi bài/đợt chỉnh sửa.

Tóm lại: **issue → branch riêng → commit → PR vào `main` với `Closes #N` → merge PR thì issue tự đóng.**

## Fidelity nội dung nguồn

Nội dung lấy từ SGK/SBT phải **chính xác tuyệt đối**.

- Agent được chọn đúng phần nào cần đưa vào `lesson`, `exercises`, `practice`, nhưng nội dung đã chọn thì không được tóm tắt, rút gọn, diễn đạt lại hoặc đổi dữ kiện.
- Giữ nguyên câu chữ, số, ký hiệu, đơn vị, điều kiện, tên riêng, chú thích và nhãn của nguồn.
- Không tự thay ví dụ nguồn bằng ví dụ tương tự.
- Không tự thêm lời gợi ý/câu hỏi dẫn dắt nhìn thấy trên bản học sinh nếu issue không yêu cầu.
- Phần agent được linh hoạt mặc định là **semantic markup và vùng trả lời**: math mode, list đúng cấu trúc nguồn, `\answerline`, `\answerline*`, `\answerbox` hoặc primitive tương đương.
- **Không tạo lời giải ẩn, đáp án ẩn hoặc comment `% Lời giải:`** nếu issue/reviewer không yêu cầu rõ ràng.
- Chỉ cần nhìn dạng câu và **ước lượng số dòng/ô học sinh cần dùng**, không cần viết đáp án ra source để quyết định khoảng trống.
- Không tự thêm nhãn trả lời như `Đáp án:`, `Thuộc A:`, `Số phần tử:` nếu nguồn không có.
- Nếu nội dung dài không vừa layout, sửa cấu trúc/style chung; không sửa chữ nguồn để làm cho vừa.
- Không âm thầm sửa typo nguồn; nếu buộc phải sửa vì lỗi làm sai kiến thức thì ghi rõ trong PR/comment.

### Heading phân khu vực của tài liệu biên soạn

Các heading cấp khu vực như `Kiến thức trọng tâm`, `Bài tập`, `Luyện tập` là **chủ ý của repository** để giáo viên/học sinh phân biệt phần bài học, bài tập SGK và luyện tập SBT. Không coi các heading này là lỗi fidelity chỉ vì nguồn không dùng đúng cùng heading.

Ngoại lệ này chỉ áp dụng cho heading phân khu vực đã được repository quy ước. Nhãn semantic nằm **bên trong lesson** như `Ví dụ`, `Câu hỏi`, `Hoạt động`, `Luyện tập 1`, `Vận dụng`, ... vẫn phải bám đúng nguồn.

## Hình ảnh: phân loại trước khi redraw

Trước khi viết TikZ, Agent phải quyết định hình thuộc loại nào.

### Có thể dựng tốt bằng TikZ

Chỉ redraw khi cấu trúc có thể mô hình hóa chính xác và kiểm chứng được, ví dụ:

- điểm, đoạn, đường, đa giác, đường tròn, trục số;
- hình học có quan hệ thẳng hàng/trung điểm/giao điểm/song song/vuông góc rõ ràng;
- sơ đồ cây, sơ đồ phân tích, lưới toán học đơn giản;
- sơ đồ/ký hiệu đơn giản mà mọi nhãn và dữ kiện có thể giữ 100%.

Khi redraw phải đọc `.agents/skills/tikz-math-figures/SKILL.md` và giữ fidelity tuyệt đối.

### Không nên redraw bằng TikZ

Không cố vẽ bản xấp xỉ cho:

- ảnh thực tế, người/vật/cảnh vật;
- tranh minh họa;
- logo, biểu tượng hoặc hình nghệ thuật có chi tiết phức tạp;
- phối cảnh, cắt-ghép, texture hoặc hình nhiều chi tiết mà không thể tái tạo chắc chắn;
- bất kỳ hình nào mà TikZ chỉ cho kết quả “trông gần giống”.

Ưu tiên asset/crop nguồn. Nếu chưa có asset phù hợp, **không bịa hình thay thế**. Đặt comment source-only ngay vị trí cần hình theo convention:

```tex
% TODO(HINH-NGUON): bo sung asset/crop dung tu nguon; khong redraw xap xi bang TikZ.
```

Có thể thêm trang/nguồn vào comment để giáo viên tìm nhanh. Comment này chỉ dành cho giáo viên/maintainer, **không được render thành lời nhắc trên bản học sinh**.

## Không hard-code style/layout rải rác trong content

Content phải ưu tiên semantic markup và primitive/style chung của repository. Agent **không tự thêm style cục bộ chỉ để làm đẹp hoặc cứu layout**.

Mặc định không vá content bằng các lệnh như:

- `\vspace`, `\hspace`, `\kern`, `\small`, `\scriptsize`, `\fontsize` tùy tiện;
- `\setlength`, margin/padding/line spacing cục bộ;
- local `itemsep`, `leftmargin`, `column-sep` chỉ để chỉnh bằng mắt;
- `\makebox`, `minipage`, `resizebox`, `scalebox` chỉ để ép vị trí/kích thước;
- cùng một TikZ style/font/spacing literal được lặp ở nhiều file thay vì đưa thành style/macro dùng chung.

Ngoại lệ hợp lệ:

- tọa độ/kích thước thực sự là dữ kiện hoặc cấu trúc của hình nguồn;
- kích thước vùng trả lời được chọn theo đúng dạng bài và theo primitive được repository cho phép;
- layout cục bộ là semantics đặc biệt của chính nguồn, không phải chỉnh thẩm mỹ tùy ý;
- skill/convention repository cho phép rõ ràng trường hợp đó.

Nếu phát hiện spacing, font, paragraph style, list spacing hoặc khoảng cách tổng thể bất hợp lý, **không sửa hàng loạt từng content file**. Báo lại và ưu tiên chỉnh ở `config/`, package config, macro hoặc style chung khi maintainer yêu cầu. Nếu content cần một loại vùng vẽ/khoảng trống semantic mới mà chưa có primitive chung, **không tự thêm primitive/config mới**. Dùng primitive hiện có trực tiếp nếu đủ; nếu thật sự cần thay đổi ngoài `content/`, dừng ở mức đề xuất và xin maintainer duyệt theo `.agents/skills/repository-change-approval/SKILL.md`.

## Numbering `questions`

Repository chỉ có **một** list `questions`. Không tạo thêm `lessonquestions`, `exercisequestions`, `practicequestions` chỉ để tách counter.

- Trong `lesson`, **không dùng `questions` và không dùng `[resume]` để sinh nhãn semantic** như `Ví dụ`, `Câu hỏi`, `Hoạt động`, `Luyện tập`, `Vận dụng`, `Thực hành`, `Thử thách nhỏ`. Viết nhãn nhìn thấy trong SGK trực tiếp bằng LaTeX thường, ví dụ `\textbf{Câu hỏi 2}`, rồi dùng `enumerate`/`itemize` thường nếu block có nhiều ý.
- Không suy nhãn semantic từ counter. Nhãn, số thứ tự và kiểu danh sách phải bám đúng SGK nguồn.
- `questions` + named series chỉ dùng cho các bài tập được đánh số trong `exercises` / `practice`.
- Trong `exercises` / `practice`, câu con chuẩn `a)`, `b)`, `c)`, ... của một bài tập phải dùng nested `questions`; chỉ giữ `enumerate` hoặc custom label khi nguồn thật sự dùng kiểu nhãn đặc biệt mà level 2 của `questions` không biểu diễn đúng.
- `exercises` / Bài tập SGK: ở bài đầu tiên của chương dùng `\begin{questions}[series=SGK]`; các bài sau trong cùng chương dùng `\begin{questions}[resume=SGK]`.
- `practice` / Luyện tập SBT: dùng named series riêng `SBT`: `\begin{questions}[series=SBT]` rồi `\begin{questions}[resume=SBT]`.
- Khi sang chương mới và numbering nguồn reset, bắt đầu lại bằng `series=SGK` / `series=SBT`.
- Dùng `resume=<tên-series>` chứ không dùng `resume*` cho SGK/SBT khi style/label do environment hiện tại quyết định.
- Không hard-code số bài tập SGK/SBT bằng `\item[1.6]`, `\sourceitem{1.6}` hoặc tự gõ `Bài 1.6.` chỉ để ép numbering.

Tóm lại: **semantic block trong `lesson` viết trực tiếp theo SGK; `questions` chỉ phục vụ numbering SGK/SBT trong `exercises`/`practice`; nội dung nguồn không được AI tự rút gọn; chỉ ước lượng và thêm vùng trả lời, không sinh lời giải ẩn.**

## Quy ước xuất từng lesson trong giai đoạn hiện tại

Việc `\sectionbreak` bắt đầu mỗi lesson trên trang mới và reset page numbering về `1` là **chủ ý hiện tại** để từng bài có thể được xuất/chia sẻ riêng khi quay video.

- Không tự bỏ `\pagenumbering{arabic}` khỏi `\sectionbreak` trong audit/refactor thông thường.
- Không coi việc nhiều lesson cùng có trang `1`, `2`, ... là bug ở giai đoạn hiện tại.
- Khi maintainer chuyển sang bản phát hành toàn sách, maintainer sẽ yêu cầu thay đổi sang page numbering liên tục; không làm trước yêu cầu đó.

## Skill chuyên biệt bắt buộc

- Mọi thay đổi ngoài `content/` → đọc `.agents/skills/repository-change-approval/SKILL.md` và chỉ thực hiện sau approval cụ thể của maintainer.
Ngoài skill biên soạn nguồn, Agent phải đọc skill chuyên biệt trước khi đụng đúng loại code sau:

- Vẽ mới, redraw, sửa hoặc audit hình toán học bằng TikZ/PGF → đọc `.agents/skills/tikz-math-figures/SKILL.md`. Chỉ dùng TikZ sau khi đã xác định hình thực sự phù hợp để redraw. Hình học phải được mô hình hóa bằng các điểm/node có tên và quan hệ semantic; không chỉ gõ tọa độ để “vẽ cho giống”. Nhãn điểm phải đọc rõ ở kích thước PDF thật.
- Dùng package `tasks`, layout lựa chọn nhiều cột, `\answerbox`, `\answerline` hoặc chỗ trống inline → đọc `.agents/skills/tasks-answer-layout/SKILL.md`. Số cột/span và kích thước vùng trả lời phải chọn theo độ dài nội dung, không dùng layout cứng.
- Dựng, sửa hoặc audit nội dung có **quan hệ hàng-cột thực sự** → đọc `.agents/skills/tabularray-math-tables/SKILL.md`. Nếu một ô `(hàng, cột)` có ý nghĩa dữ liệu, ưu tiên `tblr`/`longtblr`; không giả bảng bằng `tasks`, TikZ hoặc spacing tay. Ngược lại, lưới dùng để đếm ô/tô miền/đường đi/hình học vẫn là hình, không được biến thành bảng dữ liệu chỉ vì nó có dạng lưới.

Nếu công việc vừa lấy nội dung từ SGK/SBT vừa dùng các kỹ thuật trên thì **áp dụng đồng thời** `source-lesson-authoring` và skill chuyên biệt. Bảng có vùng trả lời thì áp dụng đồng thời table skill + answer-layout skill; cell có hình học thì áp dụng thêm TikZ skill. Fidelity nguồn luôn có ưu tiên cao hơn việc làm layout đẹp.
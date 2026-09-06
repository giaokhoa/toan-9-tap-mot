---
name: source-lesson-authoring
description: "Biên soạn bài học LaTeX từ tài liệu nguồn với yêu cầu trung thành tuyệt đối về nội dung: giữ nguyên câu chữ/dữ kiện/nhãn, phân loại đúng lesson/exercises/practice, phân loại hình trước khi redraw, chỉ ước lượng vùng trả lời phù hợp và hoàn tất công việc qua branch/PR gắn issue."
metadata:
  language: "vi"
  version: "0.7.2"
  scope: "source-materials,textbook,workbook,pdf,web,scan,latex,lesson,exercises,practice,fidelity,images,layout,github,branch,pull-request"
---

# Biên soạn bài học từ tài liệu nguồn

## Approval gate cho thay đổi ngoài `content/`

Khi làm task biên soạn/chỉnh bài, `content/` là phạm vi mặc định. **Mọi thay đổi ngoài `content/` phải được maintainer duyệt cụ thể trước khi thực hiện** và phải đọc `.agents/skills/repository-change-approval/SKILL.md`.

Không được tự ý thêm command, environment, wrapper, package config, hook, global style/spacing policy, workflow hoặc abstraction dùng chung chỉ vì thấy semantic hơn, sạch hơn, dễ chỉnh đồng loạt hơn hay để tránh lặp vài dòng. Nếu primitive hiện có đủ dùng, sử dụng trực tiếp ở đúng call-site. Nếu thật sự cần abstraction/config mới, dừng ở mức phân tích + diff mẫu và chờ maintainer duyệt cụ thể.

Dùng skill này khi issue yêu cầu tạo hoặc hoàn thiện một bài học từ SGK, SBT, PDF, ebook, ảnh scan, website hoặc tài liệu học tập khác.

## 1. Nội dung nguồn phải chính xác tuyệt đối

Agent được chọn phần nào của nguồn cần đưa vào bài và đặt vào đúng environment, nhưng phần đã chọn phải giữ nguyên nội dung nguồn.

Mặc định:

- không tóm tắt;
- không rút gọn câu chữ;
- không paraphrase hoặc đổi từ đồng nghĩa;
- không đổi số, dữ kiện, đơn vị, ký hiệu, điều kiện, tên riêng;
- không bỏ mệnh đề, chú thích, nhãn hoặc chữ trong hình;
- không gộp hai phát biểu thành một câu mới;
- không tách một phát biểu thành các bước mới có lời dẫn do AI tự viết;
- không tự thêm mẹo, giải thích, câu hỏi dẫn dắt hoặc ví dụ mới nếu issue không yêu cầu.

Được thay đổi **semantic markup** cần thiết để biểu diễn đúng nguồn, ví dụ xuống dòng source, math mode, list đúng cấu trúc, bảng thật cho dữ liệu hàng-cột và primitive vùng trả lời của repository. Không mặc định được tự thêm spacing/font/margin/style cục bộ chỉ để làm đẹp.

Nếu nội dung dài không vừa layout, sửa cấu trúc hoặc style chung; không sửa chữ nguồn để làm cho vừa.

Nếu nguồn có typo, không âm thầm sửa. Chỉ sửa khi issue/reviewer cho phép hoặc lỗi làm sai kiến thức; khi sửa phải ghi rõ trong PR/comment.

### Heading phân khu vực là ngoại lệ có chủ ý

Repository dùng các heading cấp khu vực như `Kiến thức trọng tâm`, `Bài tập`, `Luyện tập` để phân biệt mạch bài học, bài tập SGK và luyện tập SBT. Các heading này là **convention của tài liệu biên soạn**, không coi là nội dung AI tự thêm trái fidelity.

Ngoại lệ chỉ áp dụng cho heading phân khu vực đã được repository quy ước. Nhãn semantic **bên trong lesson** như `Ví dụ`, `Câu hỏi`, `Hoạt động`, `Luyện tập 1`, `Vận dụng`, ... vẫn phải đối chiếu đúng nguồn.

### Quy ước phân số bắt buộc

- **Mọi phân số hiển thị cho học sinh phải dùng `\dfrac`** để tử số và mẫu số có kích thước dễ đọc, kể cả khi phân số nằm trong inline math, display math, `lesson`, `exercises`, `practice`, ví dụ, bảng, nhãn hình/TikZ hoặc biểu thức chen trong câu văn.
- Không dùng `\frac` hoặc `\tfrac` cho phân số student-facing. Khi gặp source hiện có dùng hai lệnh này, đổi primitive LaTeX sang `\dfrac` nhưng **không đổi tử số, mẫu số, dấu, dữ kiện hoặc ý nghĩa toán học**.
- Không tự đổi literal của nguồn từ dạng có dấu gạch chéo như `1/2` sang phân số xếp tầng nếu dấu `/` là một phần wording/dữ liệu nguồn; quy ước này chỉ áp dụng khi nội dung được biểu diễn bằng primitive phân số LaTeX.
- Ngoại lệ chỉ dành cho implementation nội bộ của package/macro nếu `\frac` không trực tiếp tạo phân số student-facing. Không sửa source package bên thứ ba chỉ để thoả audit nội dung.
- Trước khi giao, audit source nội dung đang sửa và, khi phạm vi cho phép, toàn bộ `content/**/*.tex`; không để sót `\frac`/`\tfrac` trong nội dung học sinh.

## 2. Phần AI được linh hoạt: semantic markup và vùng trả lời

Mặc định agent **không viết lời giải ẩn, đáp án ẩn hoặc `% Lời giải:` trong source**. Không cần tạo lời giải đầy đủ chỉ để quyết định khoảng trống.

Agent chỉ cần nhìn dạng câu và **ước lượng lượng chữ/phép tính học sinh cần viết**, rồi chọn vùng trả lời phù hợp:

- `answerbox`: một ký hiệu, một chữ/số rất ngắn, Đ/S hoặc lựa chọn cực ngắn;
- `answerline*`: câu trả lời ngắn cần điền inline tại đúng vị trí nguồn đã chừa;
- `answerline`: một dòng trình bày độc lập;
- lặp `answerline` 2–3 lần khi câu rõ ràng cần nhiều dòng hơn;
- primitive khác: dùng chức năng tương đương theo convention repository.

Quy tắc:

- chỉ **ước lượng** số dòng cần thiết; không cần viết đáp án ra source để chứng minh ước lượng;
- không chèn nhãn trả lời do AI tự nghĩ như `Đáp án:`, `Thuộc A:`, `Số phần tử:` nếu nguồn không có;
- không sửa câu nguồn để phù hợp với ô/dòng;
- không tách câu nguồn thành nhiều bước mới chỉ để đặt nhiều ô;
- nếu nguồn đã có dấu `?` hoặc chỗ trống, đặt primitive đúng ngay vị trí đó;
- nếu nguồn không có chỗ trống, đặt vùng trả lời sau câu/ý tương ứng;
- nếu không chắc một hay hai dòng, ưu tiên chừa hơi rộng hơn thay vì thêm lời gợi ý.

Chỉ viết lời giải/đáp án khi issue hoặc reviewer yêu cầu rõ ràng.

## 3. GitHub workflow bắt buộc: issue → branch → PR

Mỗi Agent xử lý issue phải làm việc trên **branch riêng** và hoàn tất qua **pull request**. Không commit trực tiếp vào default branch.

1. Đọc issue và xác định đúng phạm vi công việc.
2. Lấy `main` hiện tại làm base rồi tạo branch riêng cho issue.
3. Mặc định một issue dùng một branch; tên branch nên chứa số issue và slug ngắn, ví dụ `lesson/issue-12-so-nguyen` hoặc `fix/issue-12-numbering`.
4. Không dùng lại branch của issue đã hoàn thành cho issue khác.
5. Commit toàn bộ thay đổi của issue trên branch đó.
6. Build/check theo convention repository trước khi giao.
7. Mở PR từ branch vào `main`.
8. Trong body PR phải có closing keyword chính xác `Closes #<issue-number>`. Chỉ `#N`, `Related to #N` hoặc comment thường **không thay thế** closing keyword.
9. Nếu PR thực sự hoàn thành nhiều issue theo yêu cầu, dùng một dòng `Closes #N` cho từng issue.
10. Không đóng issue thủ công trước merge.

**Semantics bắt buộc:** issue chỉ được coi là hoàn thành khi thay đổi đã vào `main`. Vì vậy dùng cơ chế GitHub `Closes #N`: issue tự đóng khi PR được **merge** vào default branch. Nếu PR bị đóng mà không merge, issue phải còn mở để công việc không bị mất dấu.

### CI và warning

CI của repository hiện là **build gate**, không phải warning gate tuyệt đối.

- Không tự sửa CI để biến mọi warning LaTeX thành failure.
- Build phải tạo được artifact đúng như workflow yêu cầu.
- Khi issue/reviewer yêu cầu audit warning, Agent phải đọc final LaTeX pass, phân loại warning, sửa warning thuộc scope và báo rõ warning tồn tại sẵn ngoài scope.
- Warning audit có thể được maintainer yêu cầu như một lượt riêng sau mỗi bài/đợt chỉnh sửa; không tự biến policy đó thành CI cứng.

Trước khi kết thúc lượt làm việc, Agent phải kiểm tra:

- PR có đúng base `main`;
- PR body có đúng `Closes #N`;
- branch chỉ chứa thay đổi thuộc phạm vi issue;
- CI/check cần thiết đã chạy hoặc trạng thái hiện tại được báo rõ.

## 4. Đọc conventions repository trước khi viết

Trước khi sửa nội dung:

1. đọc `AGENTS.md`, README hoặc hướng dẫn local;
2. xem một bài đã hoàn thiện gần nhất;
3. đọc macro/environment liên quan đến kiến thức, câu hỏi, bài tập, luyện tập, vùng trả lời và hình ảnh;
4. xác định asset convention;
5. xác định command build/check chính thức.

Repository hiện tại là authority cho format và tooling. Skill này là authority cho fidelity, provenance và workflow.

Không hardcode repository, path asset, command build hoặc tên environment nếu repository đã có convention khác.

### Reset page theo từng lesson hiện là chủ ý

Ở giai đoạn hiện tại, `\sectionbreak` reset số trang về `1` cho mỗi lesson để maintainer có thể xuất/chia sẻ từng bài riêng khi quay video.

- Không tự bỏ reset page numbering trong audit/refactor thông thường.
- Không coi việc nhiều lesson cùng bắt đầu từ trang `1` là bug.
- Chỉ chuyển sang page numbering liên tục khi maintainer yêu cầu chuẩn bị bản phát hành toàn sách.

## 5. Xác minh nguồn và provenance

Nguồn phải được phân vai rõ:

- nguồn kiến thức chính → kiến thức, ví dụ, minh hoạ, câu hỏi trong mạch bài;
- nguồn bài tập chính → bài tập cuối bài;
- nguồn luyện tập → bài bổ sung trong `practice`;
- nguồn minh hoạ → hình/bảng/sơ đồ.

Nếu issue quy định SGK → `exercises` và SBT → `practice`, không được trộn hai nguồn.

Trước khi lấy nội dung từ một trang/ảnh:

1. mở trang/ảnh;
2. đối chiếu môn, lớp, bộ sách, chương, số bài và tên bài;
3. xác nhận trang thực sự nằm trong phạm vi bài;
4. xác nhận vai trò của nguồn.

Nếu sai bài/sai chương/không xác minh chắc:

- dừng dùng trang đó;
- tìm đúng trang trong chính nguồn;
- không suy ra trang đúng chỉ từ pattern URL;
- nếu vẫn không xác minh được, báo blocker thay vì đoán.

### Title anchor

Khi trang bài tập không tự hiện tên/số bài:

1. tìm trang ngữ cảnh có đúng `Bài N. <Tên bài>`;
2. lần theo thứ tự trang liên tục tới vùng bài tập;
3. bảo đảm chưa đi qua heading bài kế tiếp;
4. nếu không thiết lập được title anchor, dừng và báo blocker.

## 6. Workflow nội dung

1. Đọc toàn bộ phạm vi của đúng bài trước khi viết.
2. Ghi inventory nội bộ: kiến thức, quy tắc, chú ý, ví dụ, hình, câu hỏi trong mạch bài, bài tập cuối bài, bài luyện tập.
3. Gắn mỗi item với nguồn/vị trí nguồn.
4. Phân loại:
   - `lesson`: kiến thức, ví dụ/minh hoạ và câu hỏi/luyện tập nằm trong mạch học;
   - `exercises`: bài tập trực tiếp của nguồn chính;
   - `practice`: bài tập của nguồn luyện tập.
5. Chép nội dung đã chọn đúng câu chữ/dữ kiện nguồn.
6. Chỉ sau khi nội dung đúng mới thêm vùng trả lời theo ước lượng.

Mặc định giữ đầy đủ bài tập cuối bài và bài tập của đúng bài trong nguồn luyện tập, trừ khi issue cho phép chọn tập đại diện.

Không thêm bài ngoài nguồn chỉ để tăng số lượng.

## 7. Hình ảnh: phân loại trước khi dựng

**Không bắt đầu bằng câu hỏi “vẽ TikZ thế nào?”.** Trước tiên phải quyết định hình có thực sự phù hợp để redraw bằng TikZ hay không.

### 7.1. Hình phù hợp để redraw bằng TikZ

TikZ phù hợp khi hình có cấu trúc toán học/sơ đồ rõ ràng và có thể kiểm chứng fidelity, ví dụ:

- điểm, đoạn thẳng, tia, đường thẳng, đường tròn;
- tam giác, tứ giác, đa giác, trục số;
- hình có quan hệ trung điểm, giao điểm, song song, vuông góc, đối xứng;
- sơ đồ cây, sơ đồ phân tích, lưới toán học đơn giản;
- sơ đồ/ký hiệu đơn giản mà toàn bộ nhãn, số và quan hệ có thể dựng chính xác.

Khi redraw bằng TikZ phải đọc `.agents/skills/tikz-math-figures/SKILL.md`.

### 7.2. Hình không phù hợp để redraw bằng TikZ

Không cố redraw khi hình thuộc một trong các nhóm:

- ảnh thực tế, con người, động vật, đồ vật/cảnh vật có chi tiết tự nhiên;
- tranh minh hoạ;
- logo/biểu tượng phức tạp hoặc hình nghệ thuật;
- texture, phối cảnh, ảnh cắt-ghép nhiều chi tiết;
- hình mà agent phải đoán nét, màu, vị trí, số lượng chi tiết hoặc chữ;
- hình mà TikZ chỉ tạo được bản “gần giống” chứ không chứng minh được fidelity.

Trong trường hợp này:

1. ưu tiên asset gốc;
2. nếu cần thì crop sạch đúng vùng nguồn;
3. nếu chưa có asset/crop, **không bịa hoặc đơn giản hóa hình**.

Đặt comment source-only ngay đúng vị trí cần hình để giáo viên/maintainer bổ sung sau:

```tex
% TODO(HINH-NGUON): bo sung asset/crop dung tu nguon; khong redraw xap xi bang TikZ.
```

Có thể ghi thêm nguồn/trang trong comment. Comment không được render thành placeholder hay lời nhắc nhìn thấy trên bản học sinh.

### 7.3. Nếu redraw

Nếu đã xác định hình thật sự TikZ-able:

- giữ đủ mọi chữ, nhãn, số, ký hiệu, chú thích, tên điểm, tên trục, đơn vị;
- giữ đúng dữ kiện, quan hệ, hướng mũi tên, thứ tự và số lượng đối tượng;
- **cấm rút gọn chữ trong hình** để tiết kiệm diện tích;
- cấm đổi nhãn thành từ ngắn hơn;
- cấm bỏ chú thích vì cho rằng không cần;
- cấm thay dữ kiện bằng ví dụ tương tự.

Nếu trong quá trình redraw phát hiện không thể bảo đảm 100%, dừng redraw và quay lại asset/crop/TODO nguồn; không cố hoàn thiện một hình xấp xỉ.

## 8. Không hard-code style/layout rải rác trong content

Content phải ưu tiên **semantic markup + primitive/style chung**. Agent không tự thêm style cục bộ chỉ vì thấy trang “chưa đẹp”.

Mặc định không vá content bằng:

- `\vspace`, `\hspace`, `\kern` tùy tiện;
- `\small`, `\scriptsize`, `\fontsize` để ép nội dung vừa chỗ;
- `\setlength`, line spacing, margin/padding thủ công;
- local `itemsep`, `leftmargin`, `column-sep` chỉ để chỉnh bằng mắt;
- `\makebox`, `minipage`, `resizebox`, `scalebox` chỉ để ép vị trí/kích thước;
- style TikZ/font/spacing literal giống nhau lặp ở nhiều content file.

Ngoại lệ:

- tọa độ/kích thước là dữ kiện/cấu trúc thực sự của hình nguồn;
- kích thước vùng trả lời được chọn theo dạng bài và primitive của repository;
- layout cục bộ là đặc điểm semantic đặc biệt của chính nguồn;
- skill/convention repository cho phép rõ trường hợp đó.

Nếu spacing/font/list/layout tổng thể bất hợp lý, không patch từng bài. Báo lại và ưu tiên chỉnh ở `config/`, package config, macro hoặc style chung khi maintainer yêu cầu.

Nếu cần một loại vùng làm bài mới như vùng vẽ cao cố định, lưới trả lời, khung thực hành, ... mà repo chưa có primitive semantic, ưu tiên tạo primitive dùng chung trước; **không rải `\vspace{...}` ở nhiều lesson**.

## 9. Semantic block trong `lesson`; numbering bằng named series ngoài lesson

Repository chỉ có một list `questions`; không tạo list clone chỉ để tách counter. Tuy nhiên `questions` **không phải** cơ chế sinh nhãn semantic trong `lesson`.

Với `enumitem`:

- `lesson`: viết trực tiếp nhãn nhìn thấy trong SGK bằng LaTeX thường, ví dụ `\textbf{Ví dụ}`, `\textbf{Câu hỏi 2}`, `\textbf{Hoạt động 3}`, `\textbf{Luyện tập 1}`, `\textbf{Vận dụng}`. Không dùng `\begin{questions}` và không dùng `[resume]` để suy nhãn hoặc số thứ tự;
- nếu semantic block có nhiều ý, dùng `enumerate`/`itemize` thường với `label=` phù hợp đúng SGK (`a)`, `b)`, `1.`, `2.`, ...);
- không suy số `Câu hỏi`, `Luyện tập`, `Hoạt động`, ... từ counter; đối chiếu trực tiếp nguồn và hard-code **nhãn semantic nhìn thấy**, không hard-code mã bài tập SGK/SBT;
- SGK: bài đầu chương dùng `\begin{questions}[series=SGK]`; bài sau dùng `\begin{questions}[resume=SGK]` trong `exercises`;
- SBT: bài đầu chương dùng `\begin{questions}[series=SBT]`; bài sau dùng `\begin{questions}[resume=SBT]` trong `practice`;
- trong `exercises`/`practice`, câu con chuẩn `a)`, `b)`, `c)`, ... dùng nested `questions`; chỉ dùng custom list khi nguồn thật sự có label đặc biệt;
- dùng `resume=<tên-series>` thay vì `resume*` khi style/label do environment quyết định;
- khi numbering nguồn reset ở chương mới, bắt đầu lại bằng `series=SGK` / `series=SBT`;
- câu con `a)`, `b)`, ... không làm thay đổi counter top-level.

Không hard-code mã bài tập nguồn bằng `\item[1.6]`, `\sourceitem{1.6}` hoặc tự gõ số chỉ để ép hiển thị.

Trước khi viết bài mới, mở bài trước đó trong cùng chương để xác nhận chuỗi SGK/SBT đang dừng ở đâu, đồng thời mở đúng trang SGK để xác nhận nhãn semantic của `lesson`.

## 10. Checklist trước khi giao

- [ ] Đang làm trên branch riêng của issue, không commit trực tiếp vào `main`.
- [ ] PR trỏ vào `main` và body có `Closes #<issue-number>`.
- [ ] Đúng nguồn, đúng bài, đúng provenance.
- [ ] Mọi trang/ảnh đã dùng được mở và xác minh.
- [ ] Không có nội dung bị tóm tắt/rút gọn/paraphrase.
- [ ] Mọi số, ký hiệu, đơn vị, điều kiện, tên riêng và nhãn hình khớp nguồn.
- [ ] Không có ví dụ AI tự thay cho ví dụ nguồn.
- [ ] Không có lời giải ẩn/đáp án ẩn do agent tự sinh nếu issue không yêu cầu.
- [ ] Chỉ thêm vùng trả lời dựa trên ước lượng độ dài câu trả lời.
- [ ] Không có nhãn/gợi ý trả lời AI tự thêm.
- [ ] SGK/SBT đầy đủ theo phạm vi được giao và nằm đúng environment.
- [ ] Mỗi hình đã được phân loại TikZ-able hay asset/crop/TODO trước khi dựng.
- [ ] Hình không TikZ-able không bị redraw xấp xỉ; nếu thiếu asset có `% TODO(HINH-NGUON): ...` source-only đúng vị trí.
- [ ] Hình dựng lại giữ đủ chữ/nhãn/dữ kiện; không thu gọn chữ.
- [ ] Không hard-code spacing/font/layout cục bộ chỉ để làm đẹp; style lặp lại được đưa về primitive/config chung.
- [ ] Heading phân khu vực của repository được giữ đúng convention; nhãn semantic trong lesson vẫn bám nguồn.
- [ ] Không tự bỏ reset page numbering theo lesson trong giai đoạn hiện tại.
- [ ] Trong `lesson`, semantic block viết trực tiếp theo nhãn SGK; không dùng `questions`/`resume` để sinh `Ví dụ`, `Câu hỏi`, `Hoạt động`, `Luyện tập`, `Vận dụng`, ...
- [ ] Trong `exercises`/`practice`, numbering dùng đúng `questions`, `SGK`, `SBT`; subquestion chuẩn dùng nested `questions`.
- [ ] Không hard-code mã bài tập nguồn.
- [ ] Build/check thành công.
- [ ] Nếu issue/reviewer yêu cầu audit warning, final log đã được đọc và warning được phân loại đúng scope.

## Definition of done

Một bài chỉ xong khi nội dung nguồn chính xác, semantic block trong `lesson` được viết trực tiếp đúng nhãn SGK, hình đã được phân loại và xử lý đúng chiến lược TikZ/asset/crop/TODO, SGK/SBT đúng provenance và named-series numbering, vùng trả lời được ước lượng hợp lý, content không bị rải style/layout hard-code không cần thiết, không có lời giải ẩn thừa context, bản xuất build được theo workflow, và công việc đã được đưa lên PR từ branch riêng với `Closes #N` để issue tự đóng khi merge.

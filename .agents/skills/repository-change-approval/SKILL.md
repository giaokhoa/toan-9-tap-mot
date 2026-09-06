---
name: repository-change-approval
description: "Bắt buộc review và phê duyệt cực chi tiết trước mọi thay đổi ngoài content/; cấm agent tự ý thêm abstraction, config, hook, style hoặc policy dùng chung."
metadata:
  language: "vi"
  version: "1.0.1"
  scope: "repository,config,tooling,latex,workflow,skills,approval,review"
---

# Approval gate cho thay đổi ngoài `content/`

Skill này áp dụng cho **mọi task trong repository**, không chỉ task biên soạn bài học.

## Nguyên tắc mặc định

`content/` là phạm vi mặc định agent được phép sửa khi nhiệm vụ là biên soạn, sửa nội dung hoặc chỉnh markup của bài học.

**Mọi thay đổi ở file/path nằm ngoài `content/` phải được maintainer duyệt cụ thể trước khi agent thực hiện.** Không được suy diễn quyền thay config/tooling từ một yêu cầu chung như “sửa layout”, “làm cho đẹp”, “dọn code”, “hoàn thành bài”, “fix cho gọn” hoặc “chuẩn hoá”.

Một thay đổi ngoài `content/` chỉ được xem là đã được duyệt khi maintainer hoặc issue/review chỉ rõ thay đổi cần làm, file/phạm vi cần tác động hoặc hành vi cụ thể cần sửa. Approval chỉ có hiệu lực cho đúng phạm vi đó; không mở rộng thành quyền cleanup/refactor lân cận.

## Review bắt buộc trước khi đề xuất thay đổi ngoài `content/`

Trước khi đề xuất hoặc thực hiện, agent phải phân tích và nêu rõ:

1. chính xác file/path nào dự kiến thay đổi;
2. lỗi hoặc nhu cầu thực tế nào bắt buộc phải sửa ngoài `content/`;
3. diff tối thiểu dự kiến;
4. phạm vi ảnh hưởng toàn sách/repository;
5. primitive/config hiện có đã được kiểm tra như thế nào và vì sao chưa đủ;
6. regression có thể xảy ra;
7. cách build, test và kiểm artifact/output sau sửa;
8. phương án chỉ sửa `content/` hoặc dùng primitive hiện có, nếu có.

Nếu chưa có approval rõ ràng, agent **dừng ở mức phân tích + đề xuất + diff mẫu**; không ghi thay đổi vào repo.

## `build-origin/` là invariant của repository

`build-origin/` không phải cache hoặc artifact tạm. Đây là **bằng chứng lâu dài trong Git rằng trạng thái code tổng trên `main` đã build thành công**, đồng thời lưu snapshot PDF/log/fls để maintainer có thể mở lại output sau merge và review trực quan về sau.

Agent phải giữ các nguyên tắc sau khi phân tích hoặc đề xuất thay đổi CI/build:

- `build-origin/main.pdf`, `build-origin/main.log`, `build-origin/main.fls` là output chính thức cần được bảo toàn trừ khi maintainer chỉ rõ một thiết kế thay thế tương đương;
- GitHub Actions artifact chỉ là output phụ trợ có retention hữu hạn, **không được coi là replacement cho `build-origin/`**;
- build trên PR/branch riêng chỉ là merge gate cho thay đổi của PR đó; trong môi trường nhiều agent làm song song, branch riêng không đại diện cho code tổng cuối cùng sau merge;
- snapshot `build-origin/` chính thức phải được tạo từ **post-merge `main`**, tức trạng thái tổng hợp sau khi thay đổi đã vào nhánh chính;
- trước khi publish snapshot post-merge, workflow phải bảo vệ khỏi stale build: nếu `origin/main` đã advance khỏi SHA đang build thì phải bỏ publish snapshot cũ;
- nếu branch protection/ruleset chặn cách publish hiện tại, agent phải sửa **cơ chế publish** sao cho tương thích ruleset nhưng vẫn giữ post-merge build, durable snapshot và stale-build guard; không được lấy lỗi publish làm lý do xoá `build-origin/`;
- `paths-ignore: build-origin/**`, `[skip ci]` hoặc cơ chế tương đương có thể là phần chống vòng lặp publish; không được xoá chúng chỉ vì thấy “thừa” nếu chưa chứng minh workflow vẫn không tự kích hoạt vô hạn;
- trước mọi đề xuất redesign CI/build, agent **bắt buộc phải đọc workflow hiện tại và log failure thực tế**. Không được suy từ tên step hoặc từ một run đỏ rồi đề xuất xoá cơ chế hiện có.

### Các đề xuất bị cấm mặc định

Nếu maintainer chưa yêu cầu thay đổi kiến trúc build, agent không được tự đề xuất hoặc thực hiện:

- xoá `build-origin/`;
- chỉ giữ GitHub Actions artifact thay cho snapshot trong Git;
- bỏ PDF khỏi `build-origin/`;
- publish snapshot chính thức từ branch PR thay cho post-merge `main`;
- bỏ stale-main check;
- coi một PR build xanh là bằng chứng rằng code tổng sau nhiều merge song song đã được build;
- vô hiệu hoá post-merge build chỉ để làm workflow xanh.

Nếu agent cho rằng invariant này thật sự cần thay đổi, phải trình rõ lợi ích, regression, cách giữ bằng chứng build lâu dài, cách review lại PDF, cách xử lý nhiều agent song song và cách chống stale publish, rồi chờ maintainer duyệt cụ thể.

## Cấm tự ý thêm abstraction hoặc policy dùng chung

Nếu chưa được maintainer duyệt cụ thể, agent không được tự ý:

- thêm command/macro LaTeX dùng chung;
- thêm environment/wrapper dùng chung;
- thêm package hoặc package config;
- thêm hook;
- thêm global style, spacing, margin, font, numbering hoặc layout policy;
- thêm primitive vùng trả lời/vùng vẽ chỉ để tránh lặp vài dòng trong content;
- thêm helper/config chỉ vì thấy “semantic hơn”, “sạch hơn” hoặc “dễ chỉnh đồng loạt hơn”;
- thêm/chỉnh workflow hoặc CI policy;
- thêm script/tooling/build step dùng chung;
- opportunistic refactor file ngoài phạm vi issue;
- codify một thử nghiệm cục bộ thành convention repository nếu chưa được duyệt.

**Không tạo abstraction chỉ để tránh duplication nhỏ.** Nếu primitive hiện có diễn đạt trực tiếp được yêu cầu thì ưu tiên dùng primitive đó tại đúng call-site.

## Khi maintainer yêu cầu sửa ngoài `content/`

Khi đã có approval cụ thể:

- chỉ sửa đúng file/hành vi được duyệt;
- giữ diff nhỏ nhất có thể;
- không tranh thủ cleanup khác;
- không thêm tầng abstraction mới nếu yêu cầu có thể giải quyết trực tiếp;
- audit toàn repo cho call-site/dependency bị ảnh hưởng;
- build/check theo convention repo;
- với typography/layout/TikZ/table/spacing phải kiểm artifact thật, không kết luận chỉ vì CI xanh;
- đọc final LaTeX log và báo regression thuộc scope;
- PR phải mô tả rõ thay đổi ngoài `content/` đã được duyệt và lý do.

## Ví dụ

### Không được phép

Issue chỉ yêu cầu chừa thêm chỗ để học sinh làm bài, nhưng agent tự thêm macro vùng trả lời/vùng vẽ vào config rồi migrate hàng loạt content.

Sai vì yêu cầu content/layout không đồng nghĩa với approval tạo API mới ở config.

### Được phép

Maintainer chỉ đích danh một file package config và yêu cầu tăng reported height/depth của một primitive toán học thêm một lượng cụ thể.

Đó là approval cho đúng thay đổi đó. Agent được sửa đúng file/hành vi được nêu, nhưng không được nhân tiện đổi line spacing, table padding hoặc tạo framework scope mới.

### Khi chưa chắc

Nếu agent nghĩ một command/environment chung thực sự cần thiết, phải trình:

- vấn đề;
- các call-site;
- primitive hiện có;
- diff mẫu tối thiểu;
- ảnh hưởng và verification plan;

rồi chờ maintainer duyệt trước khi tạo command/environment đó.

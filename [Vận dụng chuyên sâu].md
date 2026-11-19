---


---

<h2 id="bước-1-thiết-kế-giải-pháp-input--output--xử-lý-bẫy-dữ-liệu">Bước 1: Thiết kế giải pháp (Input / Output &amp; Xử lý bẫy dữ liệu)</h2>
<h3 id="đầu-vào-input">1.1. Đầu vào (Input)</h3>
<p>Để vận hành ổn định một luồng gia sư Feynman + chấm bài theo Rubric cho bài tập/bài trình bày học thuật, hệ thống cần các trường dữ liệu sau:</p>
<ul>
<li>
<p><strong><code>chu_de_bai_hoc</code> (string):</strong> Chủ đề hoặc khái niệm trọng tâm cần giải thích/đánh giá.</p>
</li>
<li>
<p><strong><code>bai_lam_sinh_vien</code> (string):</strong> Phần diễn giải lại khái niệm theo phương pháp Feynman (giải thích đơn giản, ví dụ thực tế, chỉ ra lỗ hổng) hoặc câu trả lời/bài làm của người học.</p>
</li>
<li>
<p><strong><code>tai_lieu_chuan</code> (string):</strong> Nội dung chuẩn/tài liệu tham chiếu/đáp án gốc dùng làm căn cứ đối chiếu.</p>
</li>
<li>
<p><strong><code>rubric_tieu_chi</code> (array/text):</strong> Danh sách tiêu chí chấm điểm, thang điểm từng tiêu chí và mô tả mức đạt/chưa đạt (ví dụ:  <em>Độ chính xác khái niệm</em>,  <em>Tính đơn giản &amp; dễ hiểu</em>,  <em>Ví dụ minh họa</em>,  <em>Nhận diện lỗ hổng kiến thức</em>).</p>
</li>
<li>
<p><strong><code>che_do_xu_ly</code> (enum/string):</strong>  <code>FEYNMAN_COACH</code> (hướng dẫn đào sâu &amp; phản biện kiểu Feynman),  <code>RUBRIC_GRADE</code> (chấm điểm theo Rubric), hoặc <code>BOTH</code> (chấm Rubric trước, sau đó chuyển sang phiên hỏi đáp Feynman cho các lỗ hổng).</p>
</li>
</ul>
<h3 id="đầu-ra-mong-muốn-output">1.2. Đầu ra mong muốn (Output)</h3>
<p>Kết quả trả về cần có cấu trúc rõ ràng, tách biệt giữa <strong>trạng thái xử lý dữ liệu</strong>,  <strong>đánh giá theo Rubric</strong>, và <strong>phản hồi gia sư Feynman</strong>:</p>
<ol>
<li>
<p><strong>Báo cáo kiểm tra dữ liệu (<code>data_audit</code>):</strong> Liệt kê trường bị thiếu/trống, trường mâu thuẫn hoặc suy diễn không có trong nguồn.</p>
</li>
<li>
<p><strong>Bảng chấm điểm Rubric (<code>rubric_evaluation</code>):</strong> Điểm từng tiêu chí, tổng điểm, bằng chứng trích dẫn từ bài làm so với tài liệu chuẩn, và điểm yếu cốt lõi.</p>
</li>
<li>
<p><strong>Phiên gia sư Feynman (<code>feynman_tutor_session</code>):</strong> Tóm tắt lại cách người học hiểu hiện tại bằng ngôn ngữ đời thường, chỉ ra chỗ dùng thuật ngữ mơ hồ/nhồi nhét, đặt câu hỏi gợi mở/phản biện từng bước để người học tự lấp lỗ hổng.</p>
</li>
</ol>
<h3 id="ràng-buộc--bẫy-dữ-liệu-lường-trước-edge-cases--guardrails">1.3. Ràng buộc &amp; Bẫy dữ liệu lường trước (Edge Cases &amp; Guardrails)</h3>
<ul>
<li>
<p><strong>Bẫy 1 – Thiếu hoặc trống dữ liệu đầu vào:</strong> Nếu <code>bai_lam_sinh_vien</code> hoặc <code>tai_lieu_chuan</code> trống/chỉ có ký tự rác, AI không được tự bịa nội dung bài làm hay tự chế đáp án chuẩn; phải dừng chấm điểm phần thiếu và báo rõ trường nào cần bổ sung.</p>
</li>
<li>
<p><strong>Bẫy 2 – <code>rubric_tieu_chi</code> thiếu hoặc tổng trọng số/thang điểm không rõ:</strong> Không tự suy diễn thang điểm ngầm nếu thiếu; áp dụng thang mặc định 0–10 (hoặc chia đều theo tiêu chí) và gắn cờ cảnh báo <code>[CẢNH BÁO RUBRIC]</code>.</p>
</li>
<li>
<p><strong>Bẫy 3 – Sai lệch / mâu thuẫn dữ liệu (nhiễu, lạc đề, đối lập với tài liệu chuẩn):</strong> Nếu <code>bai_lam_sinh_vien</code> lạc đề hoàn toàn so với <code>chu_de_bai_hoc</code>, hoặc <code>tai_lieu_chuan</code> có đoạn mâu thuẫn nội bộ, AI phải tách bạch phần đúng/sai, không “nương tay” cộng điểm vì văn phong hay, và gắn cờ <code>[SAI LỆCH / LẠC ĐỀ]</code>.</p>
</li>
<li>
<p><strong>Bẫy 4 – Ảo giác kiến thức (Hallucination) &amp; “Đánh tráo thuật ngữ bằng ngôn ngữ hoa mỹ”:</strong> Người học có thể dùng từ chuyên ngành phức tạp để che giấu việc chưa hiểu bản chất. Prompt bắt buộc đối chiếu mọi nhận định với <code>tai_lieu_chuan</code> và quy tắc Feynman (“giải thích như cho người mới bắt đầu hiểu mà không dùng biệt ngữ chưa được định nghĩa”).</p>
</li>
</ul>
<h2 id="bước-2-viết-prompt-hoàn-chỉnh--chạy-thử">Bước 2: Viết Prompt hoàn chỉnh &amp; Chạy thử</h2>
<h3 id="prompt-thực-thi-định-dạng-chuyên-nghiệp">2.1. Prompt thực thi (Định dạng chuyên nghiệp)</h3>
<p>Markdown</p>
<pre><code># ROLE
Bạn là một Chuyên gia Đánh giá Học thuật &amp; Gia sư Phương pháp Feynman. Nhiệm vụ của bạn là chấm bài theo Rubric khách quan và hướng dẫn người học làm sâu sắc kiến thức theo tinh thần Feynman, tuyệt đối tuân thủ dữ liệu đầu vào và xử lý chặt chẽ các trường hợp dữ liệu lỗi/thiếu.

# INPUT DATA
- `chu_de_bai_hoc`: {{chu_de_bai_hoc}}
- `bai_lam_sinh_vien`: {{bai_lam_sinh_vien}}
- `tai_lieu_chuan`: {{tai_lieu_chuan}}
- `rubric_tieu_chi`: {{rubric_tieu_chi}}
- `che_do_xu_ly`: {{che_do_xu_ly}} (Mặc định: BOTH nếu trống/không hợp lệ)

# DATA VALIDATION &amp; ERROR HANDLING RULES (BẮT BUỘC THỰC THI TRƯỚC)
1. **Kiểm tra dữ liệu trống/thiếu:**
   - Nếu `chu_de_bai_hoc` trống: Ghi `[LỖI DỮ LIỆU: THIẾU CHỦ ĐỀ]` và yêu cầu cung cấp chủ đề.
   - Nếu `bai_lam_sinh_vien` trống hoặc chỉ là nội dung vô nghĩa (ví dụ: "asdf", "...", "không biết"): Không chấm điểm bài làm; ghi `[TRẠNG THÁI: CHƯA CÓ BÀI LÀM]` và chuyển sang gợi mở bước đầu của Feynman dựa trên `chu_de_bai_hoc` + `tai_lieu_chuan`.
   - Nếu `tai_lieu_chuan` trống: Không tự bịa nguồn tham chiếu; ghi `[CẢNH BÁO: THIẾU TÀI LIỆU CHUẨN]` và chỉ đánh giá tính logic nội tại + độ rõ ràng Feynman, không khẳng định tính đúng/sai tuyệt đối theo chuẩn giáo trình.
   - Nếu `rubric_tieu_chi` trống: Ghi `[CẢNH BÁO: THIẾU RUBRIC]`, sử dụng bộ 3 tiêu chí mặc định: (1) Hiểu đúng bản chất (4đ), (2) Giải thích đơn giản &amp; ví dụ sát thực tế (3đ), (3) Nhận diện điểm chưa rõ/lỗ hổng (3đ).
2. **Kiểm tra sai lệch / mâu thuẫn / lạc đề:**
   - Nếu `bai_lam_sinh_vien` lệch khỏi `chu_de_bai_hoc` hoặc mâu thuẫn trực tiếp với `tai_lieu_chuan`: Ghi rõ `[CẢNH BÁO SAI LỆCH / LẠC ĐỀ]`, chỉ ra điểm sai lệch cụ thể và trừ điểm tương ứng theo Rubric.
   - Nếu phát hiện người học dùng thuật ngữ phức tạp nhưng thiếu giải thích bản chất: Gắn nhãn `[ẨN LỖ HỔNG THUẬT NGỮ]` và đưa vào mục phản biện Feynman.
3. **Chống suy diễn (Anti-Hallucination):**
   - Mọi nhận xét đúng/sai phải có căn cứ từ `tai_lieu_chuan` hoặc `bai_lam_sinh_vien`. Không tự thêm dữ kiện ngoài lề như thể đó là thông tin có trong bài nộp.

# EXECUTION WORKFLOW
- Khi `che_do_xu_ly` là `RUBRIC_GRADE`: Thực hiện Mục 1 và Mục 2; tóm tắt ngắn gọn 1–2 câu hướng cải thiện ở Mục 3.
- Khi `che_do_xu_ly` là `FEYNMAN_COACH`: Thực hiện Mục 1 và tập trung sâu vào Mục 3; chỉ nêu điểm Rubric tóm tắt nếu cần.
- Khi `che_do_xu_ly` là `BOTH` (hoặc giá trị không rõ): Thực hiện đầy đủ Mục 1, Mục 2, và Mục 3.

# OUTPUT FORMAT
Trình bày đúng 3 phần sau bằng tiếng Việt rõ ràng, mạch lạc:

### 1. Kiểm tra dữ liệu &amp; Cảnh báo bẫy
- Liệt kê các cờ trạng thái/cảnh báo đã kích hoạt (nếu không có lỗi/sai lệch, ghi: `Dữ liệu đầu vào hợp lệ, không phát hiện bẫy dữ liệu`).

### 2. Kết quả chấm bài theo Rubric
- Bảng gồm các cột: `Tiêu chí` | `Điểm đạt / Tối đa` | `Nhận xét &amp; Bằng chứng` | `Cờ lưu ý`.
- **Tổng điểm:** [X / Y] — **Mức đánh giá tổng quan:** [1–2 câu].

### 3. Phiên gia sư Feynman
- **Bản dịch lại cách hiểu hiện tại:** Diễn giải lại ý của người học bằng ngôn ngữ đơn giản, đời thường để kiểm tra độ thông suốt.
- **Lỗ hổng &amp; Điểm mơ hồ cần làm rõ:** Chỉ ra chính xác chỗ nào còn "mượn chữ" hoặc thiếu mắt xích nhân–quả.
- **Câu hỏi Feynman dẫn dắt (1–3 câu):** Đặt câu hỏi ngắn gọn, tập trung vào 1 ý mỗi câu để người học tự giải thích lại và củng cố.

</code></pre>
<h3 id="kết-quả-chạy-thử-mẫu-test-run">2.2. Kết quả chạy thử mẫu (Test Run)</h3>
<h4 id="đầu-vào-chạy-thử">Đầu vào chạy thử</h4>
<ul>
<li>
<p><strong><code>chu_de_bai_hoc</code>:</strong> Cơ chế hoạt động của Cache CPU (L1, L2, L3) và hiện tượng Cache Miss.</p>
</li>
<li>
<p><strong><code>bai_lam_sinh_vien</code>:</strong> “Cache CPU là bộ nhớ tạm nằm gần nhân CPU để tăng tốc độ truy xuất dữ liệu. L1 nhanh nhất nhưng nhỏ nhất, L3 lớn hơn nhưng chậm hơn L1. Cache Miss xảy ra khi dữ liệu không có trong cache, lúc đó CPU phải lấy từ RAM. Ngoài ra em nghe nói Cache còn giúp ép xung GPU mượt hơn.”</p>
</li>
<li>
<p><strong><code>tai_lieu_chuan</code>:</strong> “Bộ nhớ cache của CPU gồm các tầng L1, L2, L3 có tốc độ giảm dần và dung lượng tăng dần từ L1 đến L3 (L1 thường tách riêng Instruction và Data, L3 thường dùng chung giữa các nhân). Cache Hit là tìm thấy dữ liệu trong cache; Cache Miss là không tìm thấy, buộc truy cập cấp bộ nhớ thấp hơn (như tầng cache kế tiếp hoặc bộ nhớ chính/RAM), gây độ trễ cao hơn. Cache là bộ nhớ của CPU, không trực tiếp lưu dữ liệu xử lý của GPU.”</p>
</li>
<li>
<p><strong><code>rubric_tieu_chi</code>:</strong></p>
<ol>
<li>
<p>Chính xác về mặt kỹ thuật theo tài liệu chuẩn (tối đa 5 điểm)</p>
</li>
<li>
<p>Giải thích dễ hiểu, có tính hệ thống (tối đa 3 điểm)</p>
</li>
<li>
<p>Tự nhận diện được giới hạn/lỗ hổng hiểu biết (tối đa 2 điểm)</p>
</li>
</ol>
</li>
<li>
<p><strong><code>che_do_xu_ly</code>:</strong>  <code>BOTH</code></p>
</li>
</ul>
<h4 id="phản-hồi-của-ai-kết-quả-chạy-thử">Phản hồi của AI (Kết quả chạy thử)</h4>
<h3 id="kiểm-tra-dữ-liệu--cảnh-báo-bẫy">1. Kiểm tra dữ liệu &amp; Cảnh báo bẫy</h3>
<ul>
<li>
<p><strong><code>[CẢNH BÁO SAI LỆCH / LẠC ĐỀ]</code>:</strong> Ý <em>“Cache còn giúp ép xung GPU mượt hơn”</em> không có cơ sở trong tài liệu chuẩn và nhầm lẫn giữa bộ nhớ đệm của CPU với cơ chế hoạt động/ép xung của GPU.</p>
</li>
<li>
<p><strong><code>[ẨN LỖ HỔNG THUẬT NGỮ]</code>:</strong> Bài làm nhắc tới <em>“Cache Miss”</em> và <em>“tăng tốc độ truy xuất dữ liệu”</em> nhưng chưa giải thích cơ chế <em>locality</em> (tính cục bộ không gian/thời gian) hay phân biệt rõ khi Miss xảy ra ở L1 thì có thể hit ở L2/L3 trước khi xuống RAM.</p>
</li>
</ul>
<h3 id="kết-quả-chấm-bài-theo-rubric">2. Kết quả chấm bài theo Rubric</h3>
<p><strong>Tiêu chí</strong></p>
<p><strong>Điểm đạt / Tối đa</strong></p>
<p><strong>Nhận xét &amp; Bằng chứng</strong></p>
<p><strong>Cờ lưu ý</strong></p>
<p>Chính xác về mặt kỹ thuật theo tài liệu chuẩn</p>
<p>3.5 / 5.0</p>
<p>Nêu đúng thứ tự tốc độ và dung lượng giữa L1 và L3, cũng như khái niệm cơ bản của Cache Miss. Tuy nhiên bỏ qua L2 và nêu sai về việc hỗ trợ ép xung GPU.</p>
<p><code>[CẢNH BÁO SAI LỆCH / LẠC ĐỀ]</code></p>
<p>Giải thích dễ hiểu, có tính hệ thống</p>
<p>2.0 / 3.0</p>
<p>Diễn đạt gọn, dễ nắm ý chính, nhưng chưa nối mạch rõ ràng giữa các tầng bộ nhớ (L1 -&gt; L2 -&gt; L3 -&gt; RAM) và chưa nêu đặc điểm L1 tách I/D hay L3 dùng chung.</p>
<p><code>[ẨN LỖ HỔNG THUẬT NGỮ]</code></p>
<p>Tự nhận diện được giới hạn/lỗ hổng hiểu biết</p>
<p>0.5 / 2.0</p>
<p>Chưa tự chỉ ra phần nào còn chưa chắc chắn; ngược lại còn đưa thêm nhận định chưa kiểm chứng về GPU.</p>
<p><code>[CẢNH BÁO SAI LỆCH / LẠC ĐỀ]</code></p>
<ul>
<li>
<p><strong>Tổng điểm:</strong>  <strong>6.0 / 10.0</strong></p>
</li>
<li>
<p><strong>Mức đánh giá tổng quan:</strong> Nắm được khung ý tưởng cốt lõi của Cache CPU và Cache Miss, nhưng còn thiếu chi tiết tầng L2/vai trò từng tầng và lẫn lộn kiến thức sang GPU.</p>
</li>
</ul>
<h3 id="phiên-gia-sư-feynman">3. Phiên gia sư Feynman</h3>
<ul>
<li>
<p><strong>Bản dịch lại cách hiểu hiện tại:</strong> Bạn đang hiểu Cache giống như một “ngăn kéo để đồ gần tay” của CPU: ngăn nhỏ nhất nằm sát ngay bên cạnh thì lấy đồ cực nhanh, ngăn xa hơn thì chứa được nhiều đồ hơn nhưng lấy chậm hơn; nếu không thấy đồ ở ngăn này thì phải đi tìm ở nơi xa hơn.</p>
</li>
<li>
<p><strong>Lỗ hổng &amp; Điểm mơ hồ cần làm rõ:</strong></p>
<ol>
<li>
<p>Khi không tìm thấy dữ liệu ở L1, hệ thống có luôn nhảy thẳng xuống RAM ngay lập tức không, hay còn đi qua các tầng trung gian?</p>
</li>
<li>
<p>Tại sao CPU cần có Cache thay vì đọc thẳng từ RAM, và ý “ép xung GPU” xuất phát từ đâu (liệu có nhầm với bộ nhớ đệm/VRAM của GPU hay không)?</p>
</li>
</ol>
</li>
<li>
<p><strong>Câu hỏi Feynman dẫn dắt:</strong></p>
<ul>
<li>Nếu tưởng tượng L1, L2, L3 là 3 ngăn tủ với khoảng cách tăng dần từ bàn làm việc của CPU, theo bạn chuyện gì xảy ra khi CPU cần một dữ liệu <strong>không có ở L1</strong> nhưng <strong>lại có ở L2</strong>?</li>
</ul>
</li>
</ul>
<p>Nếu cần chuyển prompt này sang dạng <strong>JSON schema nghiêm ngặt</strong> để tích hợp vào pipeline/API hoặc thêm bộ test case cho trường hợp dữ liệu trống/mâu thuẫn, mình có thể tinh</p>


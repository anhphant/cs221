- ## Smoothing, Interpolation, and Backoff
	- **Có một vấn đề khi sử dụng ước lượng khả năng cực đại cho xác suất:** bất kỳ tập hợp huấn luyện hữu hạn nào cũng sẽ thiếu một số chuỗi từ tiếng Anh
	- Khi dùng **Maximum Likelihood Estimation (MLE)**, nếu một chuỗi từ **không xuất hiện trong tập huấn luyện**, thì xác suất của nó sẽ bằng **0**
	- Nhưng đây là vấn đề lớn: Corpus luôn **hữu hạn**, Trong khi ngôn ngữ thì **vô hạn và sáng tạo**.
	- **Hậu quả nghiêm trọng**: khi tính xác suất cả câu bằng cách nhân các xác suất: chỉ cần **một n-gram có xác suất 0** → Cả câu có xác suất 0.
	- **Điều này khiến:**
		- Perplexity bị vô hạn (nghịch đảo 0)
		- Mô hình đánh giá sai những câu hợp lệ
		- Không thể tổng quát hóa tốt
- ### ^^Laplace smoothing^^
	- **simplest way**
	- ^^add-one^^: mỗi đếm n-gram đặt thành 1
		- làm cho MLE lớn hơn 0
	- Laplace smoothing không đủ tốt dể dùng trong các mô hình n-gram hiện đại
	- Nhưng nó hữu ích trong việc giới thiệu nhiều khái niệm mà chúng ta thấy trong các thuật toán làm mịn khác, cung cấp một mốc tham chiếu hữu ích, và cũng là một thuật toán làm mịn thực tiễn cho các nhiệm vụ khác như phân loại văn bản (Phụ lục K).  !? *deo lien quan*
	- **unsmoothed maximum likelihood estimate:**
	-
$$P(w_i) = \frac{c_i}{N}$$	- **Laplace smoothing:**
	-
$$P_{Laplace}(w_i) = \frac{c_i + 1}{N + V}$$	- với $V$ là số từ trong vocabulary
	- **MLE trước laplace smoothing:**
	-
$$P_{MLE}(w_n | w_{n-1}) = \frac{C(w_{n-1}w_n)}{C(w_{n-1})}$$	- **MLE sau laplace smoothing:**
	-
$$P_{Laplace}(w_n | w_{n-1}) = \frac{C(w_{n-1}w_n) + 1}{C(w_{n-1}) + V} = \frac{C^*(w_{n-1}w_n)}{C(w_{n-1})}$$	- Thay vì nhìn trực tiếp vào xác suất đã smooth, ta có thể quy đổi nó về một “count mới” $C^*$ .
	- Từ biểu thức trên, suy ra:
	-
$$C^*(w_{n-1}w_n) = \frac{(C(w_{n-1}w_n) + 1) \cdot C(w_{n-1})}{C(w_{n-1}) + V}$$	- Lưu ý rằng làm mượt cộng một đã tạo ra sự thay đổi rất lớn đối với các tần số. So sánh Hình 3.8 với các tần số gốc trong Hình 3.1
		- ới các tần số gốc trong Hình 3.1
	- Khi nhìn vào **discount** d, được định nghĩa là tỷ lệ giữa các tần số mới và cũ, cho chúng ta thấy mức độ giảm đáng kể của các tần số cho mỗi từ tiền tố (chiết khấu cho bigram 'muốn một' là 0,39, trong khi chiết khấu cho 'món ăn Trung Quốc' là 0,10, gấp 10 lần)
	- Sự thay đổi mạnh mẽ này xảy ra vì quá nhiều xác suất đã được chuyển sang tất cả các giá trị bằng không.
	- “Lý do là vì add-one smoothing cộng thêm 1 cho tất cả các bigram, kể cả những bigram chưa từng xuất hiện.
	- Do vocabulary rất lớn, số lượng bigram có count bằng 0 là cực kỳ nhiều.
	- Khi cộng 1 cho tất cả chúng, ta phải lấy một lượng xác suất rất lớn từ các bigram phổ biến để phân cho những bigram chưa thấy.
	- Vì vậy các bigram quan trọng bị giảm mạnh.”
- ### ^^Add-k smoothing^^
	- ^^add-k^^: tương tự như laplace smoothing nhưng thay vì thêm 1 thì thêm k kí tự
	-
$$P^*_{Add-k}(w_n | w_{n-1}) = \frac{C(w_{n-1}w_n) + k}{C(w_{n-1}) + kV}$$	- cần 1 phương pháp để chọn k: vd chọn trên **devset**
		- **Devset (development set)** là tập dữ liệu dùng để:
		- 👉 Điều chỉnh mô hình
		- 👉 Chọn hyperparameter
		- 👉 So sánh các phiên bản model
	- Hữu ích cho 1 vài công việc nhưng không làm tốt với language modeling, tạo số liệu với phương sai kém và thường discount không phù hợp
- ### ^^Language Model Interpolation^^
	- Trong trường hợp nếu ngữ cảnh dài quá mà không có dữ liệu, 1 cách hay là ta **giảm bớt ngữ cảnh** để có nhiều dữ liệu hơn
	- Nếu không có **trigram**:
	-
$$P(w_n \mid w_{n-2} w_{n-1})$$	- Ta dùng **bigram**:
	-
$$P(w_n \mid w_{n-1})$$	- Nếu **bigram** cũng không có, ta dùng **unigram**:
	-
$$P(w_n)$$	- ^^interpolation^^:
		- Tính xác suất mới bằng cách nội suy các xác suất tri-gram, bi-gram và unigram.
		-
$$P(w_n \mid w_{n-2} w_{n-1}) = \lambda_1 P(w_n) + \lambda_2 P(w_n \mid w_{n-1}) + \lambda_3 P(w_n \mid w_{n-2} w_{n-1})$$		- với $$\sum \lambda_i = 1$$
	- **slightly more sophisticated version**:
		- Mỗi trọng số λ được tính dựa vào ngữ cảnh:
			- Nếu một bigram có rất nhiều dữ liệu,
				- → trigram dựa trên nó đáng tin hơn
				- → ta tăng trọng số cho trigram.
			- Nếu trigram hiếm,
				- → ta giảm trọng số của nó
				- → dựa nhiều hơn vào bigram hoặc unigram.
			-
$$P(w_n | w_{n-2}w_{n-1}) = \lambda_1(w_{n-2:n-1})P(w_n) + \lambda_2(w_{n-2:n-1})P(w_n | w_{n-1}) + \lambda_3(w_{n-2:n-1})P(w_n | w_{n-2}w_{n-1})$$	- chọn λ:
		- ^^held-out^^ corpus: là tập hợp dữ liệu huấn luyện bổ sung, sử dụng để xác lập các giá trị λ
		- => Chúng ta cố định các xác suất n-gram và sau đó tìm kiếm các giá trị λ mà—khi được đưa vào phương trình 3.29—sẽ cho chúng ta xác suất cao nhất của tập dữ liệu giữ lại
		- Có nhiều cách khác nhau để tìm bộ giá trị λ tối ưu này. Một cách là sử dụng thuật toán EM, một thuật toán học lặp đi lặp lại hội tụ tới các giá trị λ tối ưu cục bộ (Jelinek và Mercer, 1980).
- ### Stupid Backoff
	- ^^backoff^^: nếu n-gram có zero counts thì lùi về (n-1) gram, tiếp tục như vậy cho tới hết zero counts.
	- ^^discount^^: giảm trọng số các n-gram bậc cao để giữ lại một phần xác suất cho các n-gram bậc thấp hơn.
		- Tại sao phải cần discount -> answerr bên dưới
		- Vấn đề: Nếu giữ nguyên xác suất trigram theo MLE → tổng xác suất của tất cả các từ bằng 1 -> khi gặp trigram chưa thấy và muốn backoff xuống bigram thì không còn “phần xác suất dư” để cấp cho bigram nữa.
	- ^^stupid backoff^^:
		- Từ bỏ ý tưởng cố gắng biến mô hình ngôn ngữ thành một phân phối xác suất thực sự.
		- Nếu một n-gram bậc cao có số lượng đếm bằng không, chúng ta đơn giản quay về n-gram bậc thấp hơn, được cân bằng bằng một trọng số cố định (không phụ thuộc vào ngữ cảnh).
		-
$$P(w_i | w_{i-N+1:i-1}) =
		  \begin{cases}
		  \frac{\text{count}(w_{i-N+1:i})}{\text{count}(w_{i-N+1:i-1})} & \text{if } \text{count}(w_{i-N+1:i}) > 0 \
		  \lambda P(w_i | w_{i-N+2:i-1}) & \text{otherwise}
		  \end{cases}$$		- Phương pháp backoff kết thúc ở unigram, có điểm $$S(w) = count(w) / N$$
-
- ## *nhắc em Hưng:*
	- n-gram là gì
	- estimate probabilities by Maximum Likelihood Estimation (MLE)
	- unigram, biagram
	- perplexity
	-

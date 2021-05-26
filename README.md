# animated-polyline
![final](https://user-images.githubusercontent.com/26211549/119631247-e8053200-be39-11eb-9b5f-0892fb1d21e4.gif)

Để vẽ được 1 đường đi trên bản đồ ta phải có danh sách các toạ độ. Đường đi chúng ta muốn, chính là các đoạn thẳng nổi các điểm liền kề trên danh sách toạ độ này
<br>
<br>
Ví dụ: chúng ta có một mảng 16 điểm, và khi nối 16 điểm này lại (bằng 15 đoạn thẳng), ta có được đường đi như hình
<br>
![image](https://user-images.githubusercontent.com/26211549/119634476-fd2f9000-be3c-11eb-9b49-c3c495ce66cd.png)
![image](https://user-images.githubusercontent.com/26211549/119632148-bf316c80-be3a-11eb-845d-83c239234983.png)
<br>
<br>
Bây giờ để đường đi đó có animation khi xuất hiện, rất đơn giản: ta chỉ cần cho các đoạn thẳng xuất hiện tuần tự nhau, đoạn này xong, tiếp đến đoạn khác, không xuất hiện cùng lúc, là ta sẽ có 1 animation đơn giản
<br>
<br>
Thành quả như sau:
![ezgif com-gif-maker](https://user-images.githubusercontent.com/26211549/119640311-afb62180-be42-11eb-9b8b-4f7d248d26df.gif)

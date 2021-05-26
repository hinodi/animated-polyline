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
Code & Kết quả như sau:
<br>
```js
  componentDidMount() {
    this._animate();
  }

  _animate = () => {
    if (this._index >= this._coords.length) return;

    this.setState(
      ({ renderedCoords }) => ({ renderedCoords: [...renderedCoords, this._coords[this._index]] }),
      () => {
        this._index++;
        setTimeout(this._animate, 200);
      }
    );
  };
  
  render() {
    return (
      <MapView>
        <MapView.Polyline coordinates={this.state.renderedCoords} />
      </MapView>
    );
  }
```

![ezgif com-gif-maker](https://user-images.githubusercontent.com/26211549/119640311-afb62180-be42-11eb-9b8b-4f7d248d26df.gif)

Phân tích code một chút, ở đây chúng ta có hàm `_animate` thực hiện thêm 1 tạo độ vào `state`, sau đó đợi `200ms` và gọi lại chính mình. Từ đó ta có một vòng lặp và tất cả các điểm được thêm một cách tuần tự vào `state`<br>
<br>
Chúng ta có một kết quả có thể xem là tạm ổn, tuy nhiên chúng ta có thể làm nó tốt hơn nữa<br>
Có thể nhận thấy, chúng ta chỉ thêm tuần tự các điểm, mà chưa quan tâm đến độ dài của đoạn thẳng nói từ điểm cuối cùng đến điểm chúng ta sắp thêm vào. Điều này dẫn đến, đối với các đoạn thẳng ngắn thì animation ổn, nhưng đối với những đoạn thẳng dài, animation hoàn toàn chưa được. Do một đoạn thẳng dài xuất hiện ngay lập tức, vẫn dẫn đến hiệu ứng chưa tốt<br>
<br>
Để giải quyết vấn đề này, ta có thể chia các đoạn thẳng dài thằng các đoạn thẳng nhỏ hơn, và thực hiện `state` thêm vào các đoạn thẳng nhỏ này.<br>
Chúng ta sẽ biến đổi mảng toạ độ ban đầu, thành một mãng toạ độ mới, với khoảng cách của các điểm liền kề luôn nhỏ hơn hoặc bằng một giá trị nào đó.


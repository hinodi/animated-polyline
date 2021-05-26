![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/26211549/119646417-57365280-be49-11eb-804f-db89ba064e55.gif)
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
Code:
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
Kết quả:<br>
<br>
![ezgif com-gif-maker](https://user-images.githubusercontent.com/26211549/119640311-afb62180-be42-11eb-9b8b-4f7d248d26df.gif)

Phân tích code một chút, ở đây chúng ta có hàm `_animate` thực hiện thêm 1 tạo độ vào `state`, sau đó đợi `200ms` và gọi lại chính mình. Từ đó ta có một vòng lặp và tất cả các điểm được thêm một cách tuần tự vào `state`<br>
<br>
Chúng ta có một kết quả có thể xem là tạm ổn, tuy nhiên chúng ta có thể làm nó tốt hơn nữa<br>
Có thể nhận thấy, chúng ta chỉ thêm tuần tự các điểm, mà chưa quan tâm đến độ dài của đoạn thẳng nói từ điểm cuối cùng đến điểm chúng ta sắp thêm vào. Điều này dẫn đến, đối với các đoạn thẳng ngắn thì animation ổn, nhưng đối với những đoạn thẳng dài, animation hoàn toàn chưa được. Do một đoạn thẳng dài xuất hiện ngay lập tức, vẫn dẫn đến hiệu ứng chưa tốt<br>
<br>
Để giải quyết vấn đề này, ta có thể chia các đoạn thẳng dài thằng các đoạn thẳng nhỏ hơn, và thực hiện `state` thêm vào các đoạn thẳng nhỏ này. Chúng ta sẽ biến đổi mảng toạ độ ban đầu, thành một mãng toạ độ mới, với khoảng cách của các điểm liền kề luôn nhỏ hơn hoặc bằng một giá trị bất kỳ nào đó<br>
<br>
Code:
<br>
```js
  componentDidMount() {
    // giá trị tối đa của một đoạn thẳng
    const MAX_DISTANCE = 100;

    // lưu danh sách độ sau khi biến lỗi
    const newCoords = [];
    // ban đầu ta có toạ độ đầu tiên của danh sách toạ độ ban đầu
    newCoords.push(this._coords[0]);

    // lặp qua tất cả các toạ độ
    for (let i = 1; i < this._coords.length; i++) {
      const lastIndex = newCoords.length - 1;
      const lastCoord = newCoords[lastIndex];
      const targetCoord = this._coords[i];

      // tính khoảng cách giữa toạ độ cuối cùng của danh sách toạ độ mới và toạ độ ban đầu đang xét
      // công thức tính có thể tìm ở đây: https://www.movable-type.co.uk/scripts/latlong.html
      const distance = calculateDistanceBetweenTwoPoints(lastCoord, targetCoord); 

      // nếu khoảng cách đó bé hơn hoặc bằng giá trị tối đa thì ta không cần biến đổi, thêm toạ độ gốc và danh sách
      if (distance <= MAX_DISTANCE) {
        newCoords.push(targetCoord);
        continue;
      }
      
      // ngược lại, nếu khoảng cách đó lớn hơn giá trị tối đa thì ta cần phải biến đổi
      // chia đoạn thẳng thành các đoạn nhỏ hơn, với số phần bằng tổng khoảng cách chia cho giá trị tối đa của một đoạn
      // ví dụ ta có khoảng cách = 500 và giá trị tối đa = 100 thì ta sẽ chia thành 5 phần nhỏ hơn
      const numberOfSegments = Math.ceil(distance / MAX_DISTANCE);

      // lần lượt tính toạ độ của các đoạn thẳng nhỏ hơn
      for (let j = 1; j < numberOfSegments; j++) {
        // tính phần trăm của đoạn nhỏ thứ j
        // ví dụ ta đang tính đoạn thứ 2, thì đoạn thứ 2 sẽ có chiều dài 200 và bằng 40% tổng khoảng cách
        const fraction = (MAX_DISTANCE * j) / distance;

        // tính toạ độ dựa trên điểm đầu, điểm cuối và phần trăm vừa tính
        // công thức tính có thể tìm ở đây: https://www.movable-type.co.uk/scripts/latlong.html
        const intermediatePoint = calculateIntermediatePoint(lastCoord, targetCoord, fraction);
        
        newCoords.push(intermediatePoint);
      }

      newCoords.push(targetCoord);
    }

    this._coords = newCoords;

    this._handleDrawPath();
    this._animate();
  }
  
  _animate = () => {
    if (this._index >= this._coords.length) return;

    this.setState(
      ({ renderedCoords }) => ({ renderedCoords: [...renderedCoords, this._coords[this._index]] }),
      () => {
        this._index++;
        setTimeout(this._animate, 20);
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
Kết quả:<br>
<br>
![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/26211549/119646437-5ef5f700-be49-11eb-80f9-4ccc149ae7b6.gif)


# React Native - Hướng dẫn làm việc với polyline và animated-polyline trên Map
![final](https://user-images.githubusercontent.com/26211549/119631247-e8053200-be39-11eb-9b5f-0892fb1d21e4.gif)

Vẽ đường đi trên bản đồ là một nghiệp vụ vô cùng quan trọng các ứng dụng gọi xe, giao hàng, đặt món ăn trên thị trường hiện nay, có thể kể đến Grab, Now, Baemin, Be,… đều đang sử dụng chức năng này. Bài viết này sẽ hướng dẫn mọi người sử dụng React Native để vẽ đường đi trên bản đồ (Google Map) và tối ưu ứng dụng khi vẽ đường đi đó cùng với Animation (diễn hoạt).

## Polyline là gì?
Polyline (đường đa tuyến) trong bản đồ (Google Map) là một tập hợp bao gồm nhiều điểm (point) và các đoạn thẳng nối các điểm liền kề. Mặc định, Polyline thường không được đóng hay khép kín. Để tạo thành một polyline khép kín, điểm đầu và điểm cuối phải giống nhau.<br>
<br>
Trong Google Maps, Polyline được sử dụng để vẽ lên bản đồ nhằm diễn đạt chỉ đường, phương hướng để đi được từ điểm này đến điểm khác.<br>

## Cách vẽ Polyline trong React Native
Để vẽ được 1 đường đi trên bản đồ, ta phải có danh sách các toạ độ. Đường đi ấy được tạo thành bởi nhiều đoạn thẳng nhỏ nối các điểm liền kề trên danh sách tọa độ này.<br>
<br>
Ví dụ: chúng ta có một mảng 16 điểm, và khi nối 16 điểm này lại (bằng 15 đoạn thẳng), ta có được đường đi
Lưu ý: bài viết sử dụng [react-native](https://reactnative.dev/) và [react-native-maps](https://github.com/react-native-maps/react-native-maps) để code mẫu
<br>
Code mẫu:
```js
  render() {
    const coordinates = [
      { latitude: 10.77309, longitude: 106.69835 },
      { latitude: 10.77281, longitude: 106.69853 },
      { latitude: 10.7731, longitude: 106.69882 },
      { latitude: 10.77371, longitude: 106.69944 },
      { latitude: 10.7734, longitude: 106.69978 },
      { latitude: 10.7736, longitude: 106.7 },
      { latitude: 10.77082, longitude: 106.70122 },
      { latitude: 10.77056, longitude: 106.70618 },
      { latitude: 10.76835, longitude: 106.7056 },
      { latitude: 10.76246, longitude: 106.70838 },
      { latitude: 10.75702, longitude: 106.71797 },
      { latitude: 10.75184, longitude: 106.72482 },
      { latitude: 10.75248, longitude: 106.72661 },
      { latitude: 10.75263, longitude: 106.728 },
      { latitude: 10.75238, longitude: 106.7284 },
      { latitude: 10.74475, longitude: 106.72915 },
    ];
    return (
      <MapView>
        <MapView.Polyline coordinates={coordinates} />
      </MapView>
    );
  }
```

Kết quả:<br>
![image](https://user-images.githubusercontent.com/26211549/119632148-bf316c80-be3a-11eb-845d-83c239234983.png)


## Cách vẽ Polyline với Animation (diễn hoạt)

Bây giờ để polyline đó có animation khi xuất hiện, rất đơn giản: ta chỉ cần cho các đoạn thẳng xuất hiện tuần tự nhau, đoạn này xong, tiếp đến đoạn khác, không xuất hiện cùng lúc, là ta sẽ có 1 animation đơn giản
<br>

Code mẫu:
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
![step1](https://user-images.githubusercontent.com/26211549/119640311-afb62180-be42-11eb-9b8b-4f7d248d26df.gif)

Phân tích code một chút, ở đây chúng ta có hàm `_animate` thực hiện thêm 1 toạ độ vào `state`, sau đó đợi `200ms` và gọi lại chính mình. Từ đó ta có một vòng lặp và tất cả các điểm được thêm một cách tuần tự vào `state`. Kết quả nhận được có thể xem là tạm ổn, tuy nhiên chúng ta có thể làm nó tốt hơn nữa<br>
<br>
Có thể nhận thấy, chúng ta chỉ thêm tuần tự các điểm, mà chưa quan tâm đến `độ dài` của đoạn thẳng nối từ điểm cuối cùng đến điểm chúng ta sắp thêm vào.<br>
Ví thế, đối với các `đoạn thẳng ngắn thì animation ổn`, nhưng đối với những đoạn thẳng dài thì chưa được vì chúng xuất hiện ngay lập tức<br>
<br>
Để giải quyết vấn đề này, ta có thể `chia các đoạn thẳng dài thành các đoạn thẳng nhỏ hơn`, và thay thế một đoạn dài bằng nhiều đoạn nhỏ tương ứng<br>
Chúng ta sẽ biến đổi mảng toạ độ ban đầu, thành một mãng toạ độ mới, với khoảng cách của các điểm liền kề luôn nhỏ hơn hoặc bằng một giá trị bất kỳ nào đó<br>
<br>
Code React Native mẫu sẽ như sau:
```js
  componentDidMount() {
    // giá trị tối đa của một đoạn thẳng
    const MAX_DISTANCE = 100;

    // lưu danh sách độ sau khi biến đổi
    const newCoords = [];
    // ban đầu ta có toạ độ đầu tiên của danh sách toạ độ mới
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
![step2](https://user-images.githubusercontent.com/26211549/119646437-5ef5f700-be49-11eb-80f9-4ccc149ae7b6.gif)
<br>
Kết thúc phần 1.<br>
Phần tiếp theo sẽ là cách để chúng ta có được animation mượt hơn nữa, và có thể áp dụng các hàm easing vào animation

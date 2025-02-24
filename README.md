<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tính Khối Lượng Bê Tông Theo Hạng Mục</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
            max-width: 800px;
            margin: auto;
        }
        h1, h2 {
            text-align: center;
        }
        .input-group {
            margin: 15px 0;
        }
        .input-group-flex {
            display: flex;
            justify-content: space-between;
        }
        input {
            margin: 5px 0;
            padding: 10px;
            width: 100%;
            box-sizing: border-box;
            font-size: 1.2em;
        }
        .dimensions {
            display: flex;
            justify-content: space-between;
        }
        .dimensions input {
            width: 32%;
            padding: 12px;
            font-size: 1.2em;
        }
        button {
            padding: 12px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            width: 100%;
            margin-top: 10px;
            font-size: 1.2em;
        }
        button:hover {
            background-color: #45a049;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: center;
            font-size: 1.2em;
        }
        th {
            background-color: #f2f2f2;
        }
        #totalRow {
            font-weight: bold;
        }
        #currentDate {
            text-align: center;
            margin: 10px 0;
            font-size: 1.2em;
        }
    </style>
</head>
<body>
    <h1>Tính Khối Lượng Bê Tông</h1>
    
    <div id="currentDate"></div> <!-- Phần tử hiển thị ngày giờ -->
    
    <h2>Nhập Thông Tin Công Trình</h2>
    
    <div class="input-group-flex">
        <div class="input-group" style="flex: 1; margin-right: 10px;">
            <label for="projectName">Tên công trình:</label>
            <input type="text" id="projectName" placeholder="Nhập tên công trình">
        </div>
        <div class="input-group" style="flex: 1;">
            <label for="phoneNumber">Số điện thoại:</label>
            <input type="text" id="phoneNumber" placeholder="Nhập số điện thoại">
        </div>
    </div>
    
    <div class="input-group">
        <label for="itemName">Tên hạng mục:</label>
        <input type="text" id="itemName" placeholder="Nhập tên hạng mục">
    </div>
    
    <div class="input-group dimensions">
        <div>
            <label>Chiều dài (m):</label>
            <input type="number" class="length" placeholder="Nhập chiều dài">
        </div>
        <div>
            <label>Chiều rộng (m):</label>
            <input type="number" class="width" placeholder="Nhập chiều rộng">
        </div>
        <div>
            <label>Chiều cao (m):</label>
            <input type="number" class="height" placeholder="Nhập chiều cao">
        </div>
    </div>

    <button onclick="addItem()">Thêm Hạng Mục</button>
    <button onclick="updateItem()" style="display: none;" id="updateButton">Cập Nhật Hạng Mục</button>
    
    <h2>Danh sách hạng mục:</h2>
    <table>
        <thead>
            <tr>
                <th>Tên Hạng Mục</th>
                <th>Chiều Dài (m)</th>
                <th>Chiều Rộng (m)</th>
                <th>Chiều Cao (m)</th>
                <th>Khối Lượng (m³)</th>
                <th>Thao Tác</th>
            </tr>
        </thead>
        <tbody id="itemList"></tbody>
        <tfoot>
            <tr id="totalRow">
                <td colspan="4">Tổng Khối Lượng:</td>
                <td id="totalVolume">0.00</td>
            </tr>
        </tfoot>
    </table>

    <button onclick="exportToExcel()">Tải Về Excel</button>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
    <script>
        const items = []; // Danh sách hạng mục trong bộ nhớ tạm thời
        let currentIndex = -1; // Chỉ số của hạng mục hiện tại đang được sửa

        // Hiển thị ngày giờ hiện tại
        function updateCurrentDate() {
            const currentDate = new Date();
            const formattedDate = currentDate.toLocaleString('vi-VN', {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit',
                hour12: false
            });
            document.getElementById('currentDate').innerText = `Ngày giờ: ${formattedDate}`;
        }

        function displayItems() {
            const itemList = document.getElementById('itemList');
            itemList.innerHTML = ''; // Xóa danh sách hiện tại
            items.forEach((item, index) => {
                const row = `<tr>
                                <td>${item.name}</td>
                                <td>${item.length}</td>
                                <td>${item.width}</td>
                                <td>${item.height}</td>
                                <td>${item.volume.toFixed(2)}</td>
                                <td>
                                    <button onclick="editItem(${index})">Sửa</button>
                                </td>
                             </tr>`;
                itemList.innerHTML += row;
            });
            updateTotalVolume();
        }

        function addItem() {
            const projectName = document.getElementById('projectName').value;
            const phoneNumber = document.getElementById('phoneNumber').value;
            const itemName = document.getElementById('itemName').value;
            const length = parseFloat(document.querySelector('.length').value);
            const width = parseFloat(document.querySelector('.width').value);
            const height = parseFloat(document.querySelector('.height').value);
            
            if (!projectName) {
                alert("Vui lòng nhập tên công trình.");
                return;
            }
            if (!itemName) {
                alert("Vui lòng nhập tên hạng mục.");
                return;
            }
            if (isNaN(length) || isNaN(width) || isNaN(height)) {
                alert("Vui lòng nhập đầy đủ số liệu.");
                return;
            }
            
            const volume = length * width * height;
            const newItem = { name: itemName, length, width, height, volume };
            
            // Nếu đang sửa, cập nhật hạng mục
            if (currentIndex >= 0) {
                items[currentIndex] = newItem;
                currentIndex = -1; // Đặt lại chỉ số
                document.getElementById('updateButton').style.display = 'none'; // Ẩn nút cập nhật
            } else {
                items.push(newItem); // Lưu vào danh sách hạng mục tạm thời
            }
            
            displayItems();
            resetFields();
        }

        function editItem(index) {
            const item = items[index];
            document.getElementById('itemName').value = item.name;
            document.querySelector('.length').value = item.length;
            document.querySelector('.width').value = item.width;
            document.querySelector('.height').value = item.height;
            currentIndex = index; // Ghi nhận chỉ số của hạng mục đang sửa
            document.getElementById('updateButton').style.display = 'block'; // Hiện nút cập nhật
        }

        function updateItem() {
            addItem(); // Gọi lại hàm thêm nhưng nó sẽ cập nhật
        }

        function resetFields() {
            document.getElementById('itemName').value = '';
            document.querySelector('.length').value = '';
            document.querySelector('.width').value = '';
            document.querySelector('.height').value = '';
        }

        function updateTotalVolume() {
            const total = items.reduce((acc, curr) => acc + curr.volume, 0);
            document.getElementById('totalVolume').innerText = total.toFixed(2);
        }

        function exportToExcel() {
            const projectName = document.getElementById('projectName').value || 'DuAn';
            const phoneNumber = document.getElementById('phoneNumber').value || 'Chưa có số điện thoại';
            const currentDate = new Date();
            const formattedDate = currentDate.toLocaleString('vi-VN', {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit',
                hour12: false
            });

            const wsData = [
                ["Tên công trình", projectName],
                ["Số điện thoại", phoneNumber],
                ["Ngày giờ", formattedDate],
                [], // Dòng trống
                ["Tên Hạng Mục", "Chiều Dài (m)", "Chiều Rộng (m)", "Chiều Cao (m)", "Khối Lượng (m³)"], // Dòng tiêu đề bảng
                ...items.map(item => [
                    item.name,
                    item.length,
                    item.width,
                    item.height,
                    item.volume.toFixed(2)
                ]),
                [], // Dòng trống
                ["Tổng Khối Lượng:", "", "", "", items.reduce((acc, curr) => acc + curr.volume, 0).toFixed(2)] // Dòng tổng khối lượng
            ];

            const ws = XLSX.utils.aoa_to_sheet(wsData);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "Hạng Mục");

            XLSX.writeFile(wb, `${projectName}.xlsx`);
        }

        // Cập nhật ngày giờ khi tải trang
        updateCurrentDate();
    </script>
</body>
</html>

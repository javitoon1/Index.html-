<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Escaner de Gastos Móvil</title>
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4/dist/tesseract.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        /* Estilos optimizados para móvil */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 15px;
            background-color: #f0f2f5;
            color: #333;
        }
        
        .container {
            max-width: 100%;
            margin: 0 auto;
        }
        
        h1 {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 5px;
        }
        
        .subtitle {
            text-align: center;
            color: #7f8c8d;
            margin-bottom: 20px;
        }
        
        .card {
            background: white;
            border-radius: 10px;
            padding: 15px;
            margin-bottom: 20px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        
        .btn {
            display: block;
            width: 100%;
            padding: 12px;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
            font-size: 16px;
            margin: 10px 0;
            cursor: pointer;
            text-align: center;
        }
        
        .btn:active {
            background: #2980b9;
        }
        
        .btn-secondary {
            background: #7f8c8d;
        }
        
        .btn-success {
            background: #27ae60;
        }
        
        #cameraView {
            width: 100%;
            height: 250px;
            background: #2c3e50;
            border-radius: 5px;
            margin-bottom: 15px;
            display: none;
        }
        
        #cameraView video {
            width: 100%;
            height: 100%;
            object-fit: cover;
            border-radius: 5px;
        }
        
        #captureBtn {
            display: none;
            width: 60px;
            height: 60px;
            border-radius: 50%;
            background: white;
            border: 4px solid #ecf0f1;
            position: fixed;
            bottom: 30px;
            left: calc(50% - 30px);
            box-shadow: 0 2px 10px rgba(0,0,0,0.3);
        }
        
        #imagePreview {
            width: 100%;
            border-radius: 5px;
            margin-bottom: 15px;
            display: none;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
        }
        
        .result-section {
            display: none;
        }
        
        .expense-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
        }
        
        .expense-table th, .expense-table td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        
        .expense-table th {
            background-color: #f2f2f2;
        }
        
        .summary {
            display: flex;
            justify-content: space-between;
            margin-top: 20px;
            padding: 15px;
            background: #ecf0f1;
            border-radius: 5px;
            font-weight: bold;
        }
        
        .loader {
            display: none;
            text-align: center;
            padding: 20px;
        }
        
        .loader-spinner {
            border: 5px solid #f3f3f3;
            border-top: 5px solid #3498db;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
            margin: 0 auto 15px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .notification {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: #2c3e50;
            color: white;
            padding: 12px 20px;
            border-radius: 50px;
            display: none;
            z-index: 1000;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Escaner de Gastos Móvil</h1>
        <p class="subtitle">Escanea tus recibos y organiza tus gastos</p>
        
        <div class="card">
            <h2>Escanear Recibo</h2>
            <button id="startCameraBtn" class="btn">Activar Cámara</button>
            <button id="uploadImageBtn" class="btn btn-secondary">Subir Imagen</button>
            <input type="file" id="fileInput" accept="image/*" style="display:none">
            
            <div id="cameraView">
                <video id="videoElement" autoplay playsinline></video>
            </div>
            <button id="captureBtn"></button>
            
            <img id="imagePreview" alt="Vista previa">
            <canvas id="canvasElement" style="display:none"></canvas>
        </div>
        
        <div class="card" id="formSection" style="display:none">
            <h2>Información del Gasto</h2>
            
            <div class="form-group">
                <label for="amount">Monto:</label>
                <input type="number" id="amount" step="0.01" placeholder="0.00">
            </div>
            
            <div class="form-group">
                <label for="date">Fecha:</label>
                <input type="date" id="date">
            </div>
            
            <div class="form-group">
                <label for="category">Categoría:</label>
                <select id="category">
                    <option value="comida">Comida</option>
                    <option value="transporte">Transporte</option>
                    <option value="compras">Compras</option>
                    <option value="otros">Otros</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="description">Descripción:</label>
                <input type="text" id="description" placeholder="Descripción del gasto">
            </div>
            
            <button id="saveExpenseBtn" class="btn btn-success">Guardar Gasto</button>
            <button id="rescanBtn" class="btn btn-secondary">Escanear de Nuevo</button>
        </div>
        
        <div class="loader" id="loader">
            <div class="loader-spinner"></div>
            <p>Procesando imagen...</p>
        </div>
        
        <div class="card result-section" id="resultSection">
            <h2>Resultados</h2>
            <div id="extractedText"></div>
            
            <button id="downloadBtn" class="btn">Descargar Excel</button>
            <button id="clearDataBtn" class="btn btn-secondary">Borrar Datos</button>
            
            <div class="summary">
                <span>Total de gastos:</span>
                <span id="totalAmount">$0.00</span>
            </div>
            
            <table class="expense-table">
                <thead>
                    <tr>
                        <th>Fecha</th>
                        <th>Descripción</th>
                        <th>Categoría</th>
                        <th>Monto</th>
                    </tr>
                </thead>
                <tbody id="expenseTableBody">
                    <!-- Los gastos se insertarán aquí -->
                </tbody>
            </table>
        </div>
    </div>
    
    <div class="notification" id="notification"></div>

    <script>
        // Variables globales
        let stream = null;
        let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
        
        // Elementos del DOM
        const videoElement = document.getElementById('videoElement');
        const canvasElement = document.getElementById('canvasElement');
        const imagePreview = document.getElementById('imagePreview');
        const captureBtn = document.getElementById('captureBtn');
        const startCameraBtn = document.getElementById('startCameraBtn');
        const uploadImageBtn = document.getElementById('uploadImageBtn');
        const fileInput = document.getElementById('fileInput');
        const formSection = document.getElementById('formSection');
        const loader = document.getElementById('loader');
        const resultSection = document.getElementById('resultSection');
        const extractedText = document.getElementById('extractedText');
        const saveExpenseBtn = document.getElementById('saveExpenseBtn');
        const rescanBtn = document.getElementById('rescanBtn');
        const downloadBtn = document.getElementById('downloadBtn');
        const clearDataBtn = document.getElementById('clearDataBtn');
        const amountInput = document.getElementById('amount');
        const dateInput = document.getElementById('date');
        const categoryInput = document.getElementById('category');
        const descriptionInput = document.getElementById('description');
        const expenseTableBody = document.getElementById('expenseTableBody');
        const totalAmountElement = document.getElementById('totalAmount');
        const notification = document.getElementById('notification');
        const cameraView = document.getElementById('cameraView');
        
        // Establecer fecha actual por defecto
        dateInput.valueAsDate = new Date();
        
        // Inicializar la aplicación
        function init() {
            updateExpenseTable();
            
            // Verificar si hay datos guardados
            if (expenses.length > 0) {
                resultSection.style.display = 'block';
            }
        }
        
        // Iniciar la cámara
        async function startCamera() {
            try {
                stopCamera(); // Detener cualquier stream existente
                cameraView.style.display = 'block';
                stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: 'environment' } 
                });
                videoElement.srcObject = stream;
                captureBtn.style.display = 'block';
            } catch (error) {
                console.error('Error al acceder a la cámara:', error);
                showNotification('No se pudo acceder a la cámara. Asegúrate de dar los permisos necesarios.');
            }
        }
        
        // Detener la cámara
        function stopCamera() {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                stream = null;
            }
            captureBtn.style.display = 'none';
            cameraView.style.display = 'none';
        }
        
        // Capturar imagen desde la cámara
        function captureImage() {
            const context = canvasElement.getContext('2d');
            canvasElement.width = videoElement.videoWidth;
            canvasElement.height = videoElement.videoHeight;
            context.drawImage(videoElement, 0, 0, canvasElement.width, canvasElement.height);
            
            // Convertir a data URL y mostrar vista previa
            const imageDataUrl = canvasElement.toDataURL('image/png');
            imagePreview.src = imageDataUrl;
            imagePreview.style.display = 'block';
            
            // Detener la cámara
            stopCamera();
            
            // Procesar la imagen
            processImage(imageDataUrl);
        }
        
        // Procesar la imagen con OCR
        async function processImage(imageData) {
            loader.style.display = 'block';
            formSection.style.display = 'none';
            resultSection.style.display = 'none';
            
            try {
                const { data: { text } } = await Tesseract.recognize(
                    imageData,
                    'spa', // Idioma español
                    { logger: m => console.log(m) }
                );
                
                extractedText.innerHTML = `<h3>Texto detectado:</h3><p>${text.replace(/\n/g, '<br>')}</p>`;
                
                // Intentar extraer información automáticamente
                extractInformation(text);
                
                loader.style.display = 'none';
                formSection.style.display = 'block';
                
            } catch (error) {
                console.error('Error en OCR:', error);
                loader.style.display = 'none';
                showNotification('Error al procesar la imagen. Intenta de nuevo.');
            }
        }
        
        // Extraer información del texto
        function extractInformation(text) {
            // Buscar montos (patrones comunes de precios)
            const amountMatches = text.match(/\$\d+[\.,]?\d*/g) || text.match(/\d+[\.,]?\d*\s*(USD|EUR|€)/g) || [];
            if (amountMatches.length > 0) {
                // Tomar el último monto (generalmente el total)
                const amount = amountMatches[amountMatches.length - 1].replace(/[^\d,.]/g, '').replace(',', '.');
                amountInput.value = parseFloat(amount).toFixed(2);
            }
            
            // Buscar fechas
            const dateMatches = text.match(/\d{1,2}[\/\-]\d{1,2}[\/\-]\d{2,4}/g) || [];
            if (dateMatches.length > 0) {
                // Intentar parsear la fecha
                const dateStr = dateMatches[0];
                const parts = dateStr.split(/[\/\-]/);
                if (parts.length === 3) {
                    let day = parts[0];
                    let month = parts[1];
                    const year = parts[2].length === 2 ? `20${parts[2]}` : parts[2];
                    
                    // Formatear a YYYY-MM-DD
                    if (day.length === 1) day = `0${day}`;
                    if (month.length === 1) month = `0${month}`;
                    
                    dateInput.value = `${year}-${month}-${day}`;
                }
            }
            
            // Buscar descripción (primeras líneas con texto)
            const lines = text.split('\n').filter(line => line.trim().length > 5);
            if (lines.length > 0) {
                descriptionInput.value = lines[0].substring(0, 50); // Limitar a 50 caracteres
            }
        }
        
        // Guardar gasto
        function saveExpense() {
            const amount = parseFloat(amountInput.value);
            const date = dateInput.value;
            const category = categoryInput.value;
            const description = descriptionInput.value;
            
            if (!amount || !date || !description) {
                showNotification('Por favor, completa todos los campos obligatorios.');
                return;
            }
            
            const expense = {
                id: Date.now(),
                amount,
                date,
                category,
                description
            };
            
            expenses.push(expense);
            localStorage.setItem('expenses', JSON.stringify(expenses));
            
            updateExpenseTable();
            formSection.style.display = 'none';
            resultSection.style.display = 'block';
            
            showNotification('Gasto guardado correctamente.');
        }
        
        // Actualizar tabla de gastos
        function updateExpenseTable() {
            expenseTableBody.innerHTML = '';
            let total = 0;
            
            expenses.forEach(expense => {
                total += expense.amount;
                
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${expense.date}</td>
                    <td>${expense.description}</td>
                    <td>${expense.category}</td>
                    <td>$${expense.amount.toFixed(2)}</td>
                `;
                
                expenseTableBody.appendChild(row);
            });
            
            totalAmountElement.textContent = `$${total.toFixed(2)}`;
        }
        
        // Descargar Excel
        function downloadExcel() {
            const worksheet = XLSX.utils.json_to_sheet(expenses);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "Gastos");
            XLSX.writeFile(workbook, "gastos.xlsx");
            
            showNotification('Archivo Excel descargado.');
        }
        
        // Borrar datos
        function clearData() {
            if (confirm('¿Estás seguro de que quieres borrar todos los datos?')) {
                expenses = [];
                localStorage.removeItem('expenses');
                updateExpenseTable();
                resultSection.style.display = 'none';
                showNotification('Datos borrados correctamente.');
            }
        }
        
        // Mostrar notificación
        function showNotification(message) {
            notification.textContent = message;
            notification.style.display = 'block';
            
            setTimeout(() => {
                notification.style.display = 'none';
            }, 3000);
        }
        
        // Event Listeners
        startCameraBtn.addEventListener('click', startCamera);
        
        captureBtn.addEventListener('click', () => {
            captureImage();
        });
        
        uploadImageBtn.addEventListener('click', () => {
            fileInput.click();
        });
        
        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length > 0) {
                const file = e.target.files[0];
                const reader = new FileReader();
                
                reader.onload = (event) => {
                    imagePreview.src = event.target.result;
                    imagePreview.style.display = 'block';
                    processImage(event.target.result);
                };
                
                reader.readAsDataURL(file);
            }
        });
        
        saveExpenseBtn.addEventListener('click', saveExpense);
        
        rescanBtn.addEventListener('click', () => {
            imagePreview.style.display = 'none';
            formSection.style.display = 'none';
            startCamera();
        });
        
        downloadBtn.addEventListener('click', downloadExcel);
        
        clearDataBtn.addEventListener('click', clearData);
        
        // Inicializar la aplicación
        init();
    </script>
</body>
</html>

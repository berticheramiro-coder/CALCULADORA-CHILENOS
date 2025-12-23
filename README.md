# CALCULADORA-CHILENOS
CALCULAR PESOS CHILENOS A ARG
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Calculadora de Divisas</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <style>
        body {
            font-family: Arial, Helvetica, sans-serif;
            background: #f4f6f8;
            margin: 0;
            padding: 20px;
        }

        .container {
            max-width: 650px;
            margin: auto;
            background: white;
            padding: 25px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        h1 {
            text-align: center;
            margin-bottom: 25px;
        }

        label {
            font-weight: bold;
            margin-top: 15px;
            display: block;
        }

        input {
            width: 100%;
            padding: 10px;
            margin-top: 5px;
            border-radius: 6px;
            border: 1px solid #ccc;
            font-size: 16px;
        }

        .result {
            background: #f0f0f0;
            padding: 12px;
            border-radius: 6px;
            margin-top: 10px;
            font-size: 16px;
        }

        .highlight {
            font-weight: bold;
        }

        button {
            margin-top: 15px;
            padding: 12px;
            width: 100%;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            cursor: pointer;
            background: #007bff;
            color: white;
        }

        button:hover {
            background: #0056b3;
        }

        .history {
            margin-top: 35px;
        }

        .purchase {
            background: #fafafa;
            padding: 12px;
            border-radius: 6px;
            margin-bottom: 10px;
            border-left: 4px solid #007bff;
        }

        .delete {
            color: red;
            cursor: pointer;
            float: right;
            font-weight: bold;
        }

        .totals {
            background: #e9ecef;
            padding: 12px;
            border-radius: 6px;
            margin-top: 15px;
            font-weight: bold;
        }

        .small {
            font-size: 14px;
            color: #555;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🧮 Calculadora de Divisas</h1>

    <label>Precio Chileno (CLP)</label>
    <input type="number" id="precioCLP" placeholder="Ej: 150000">

    <label>Descuento (%)</label>
    <input type="number" id="descuento" placeholder="Ej: 10">

    <div class="result">
        <div>Precio final CLP: <span class="highlight" id="finalCLP">0</span></div>
        <div>Ahorro por descuento: <span class="highlight" id="ahorro">0</span></div>
        <hr>
        <div>Precio en USD: <span class="highlight" id="precioUSD">0</span></div>
        <div>Precio en ARS: <span class="highlight" id="precioARS">0</span></div>
    </div>

    <label>Descripción de la compra</label>
    <input type="text" id="descripcion" placeholder="Ej: Amazon, AliExpress, etc.">

    <button onclick="guardarCompra()">Guardar compra</button>

    <div class="history">
        <h2>📋 Historial de compras</h2>
        <div id="listaCompras"></div>

        <div class="totals">
            Total compras: <span id="totalCompras">0</span><br>
            Total USD: <span id="totalUSD">0</span><br>
            Total ARS: <span id="totalARS">0</span>
        </div>

        <button style="background:#dc3545" onclick="borrarTodo()">Borrar todo</button>
    </div>
</div>

<script>
    // 🔧 CONFIGURACIÓN (podés cambiar estos valores)
    const TIPO_CAMBIO_USD = 900;
    const TIPO_CAMBIO_ARS = 1.66;

    const precioCLPInput = document.getElementById("precioCLP");
    const descuentoInput = document.getElementById("descuento");

    precioCLPInput.addEventListener("input", calcular);
    descuentoInput.addEventListener("input", calcular);

    function calcular() {
        let clp = parseFloat(precioCLPInput.value) || 0;
        let descuento = parseFloat(descuentoInput.value) || 0;

        if (descuento > 100) descuento = 100;
        if (descuento < 0) descuento = 0;

        let ahorro = clp * (descuento / 100);
        let finalCLP = clp - ahorro;

        let usd = finalCLP / TIPO_CAMBIO_USD;
        let ars = finalCLP * TIPO_CAMBIO_ARS;

        document.getElementById("finalCLP").innerText = finalCLP.toFixed(2);
        document.getElementById("ahorro").innerText = ahorro.toFixed(2);
        document.getElementById("precioUSD").innerText = usd.toFixed(2);
        document.getElementById("precioARS").innerText = ars.toFixed(2);
    }

    function guardarCompra() {
        let clp = parseFloat(precioCLPInput.value);
        if (!clp || clp <= 0) {
            alert("Ingresá un precio válido");
            return;
        }

        let descuento = parseFloat(descuentoInput.value) || 0;
        let descripcion = document.getElementById("descripcion").value || "Sin descripción";

        let ahorro = clp * (descuento / 100);
        let finalCLP = clp - ahorro;

        let compra = {
            fecha: new Date().toLocaleString(),
            clp: clp,
            descuento: descuento,
            usd: (finalCLP / TIPO_CAMBIO_USD).toFixed(2),
            ars: (finalCLP * TIPO_CAMBIO_ARS).toFixed(2),
            descripcion: descripcion
        };

        let compras = JSON.parse(localStorage.getItem("compras")) || [];
        compras.push(compra);
        localStorage.setItem("compras", JSON.stringify(compras));

        document.getElementById("descripcion").value = "";
        cargarCompras();
    }

    function cargarCompras() {
        let compras = JSON.parse(localStorage.getItem("compras")) || [];
        let lista = document.getElementById("listaCompras");
        lista.innerHTML = "";

        let totalUSD = 0;
        let totalARS = 0;

        compras.forEach((c, index) => {
            totalUSD += parseFloat(c.usd);
            totalARS += parseFloat(c.ars);

            lista.innerHTML += `
                <div class="purchase">
                    <span class="delete" onclick="eliminarCompra(${index})">✖</span>
                    <div><strong>${c.descripcion}</strong></div>
                    <div class="small">${c.fecha}</div>
                    <div>CLP: ${c.clp}</div>
                    <div>Descuento: ${c.descuento}%</div>
                    <div>USD: ${c.usd}</div>
                    <div>ARS: ${c.ars}</div>
                </div>
            `;
        });

        document.getElementById("totalCompras").innerText = compras.length;
        document.getElementById("totalUSD").innerText = totalUSD.toFixed(2);
        document.getElementById("totalARS").innerText = totalARS.toFixed(2);
    }

    function eliminarCompra(index) {
        let compras = JSON.parse(localStorage.getItem("compras")) || [];
        compras.splice(index, 1);
        localStorage.setItem("compras", JSON.stringify(compras));
        cargarCompras();
    }

    function borrarTodo() {
        if (confirm("¿Seguro que querés borrar todo el historial?")) {
            localStorage.removeItem("compras");
            cargarCompras();
        }
    }

    cargarCompras();
</script>

</body>
</html>

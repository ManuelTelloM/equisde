##  Información del Estudiante

- **Nombre:** [Manuel Jesus Tello May]
- **Matrícula:** [SW2509054]
- **Grupo:** [B]
- **Cuatrimestre:** Primer Cuatrimestre
- **Carrera:** TSU en Desarrollo e Innovación de Software
- **Profesor:** Jorge Javier Pedrozo Romero

---

##  Descripción del Proyecto

Este repositorio contiene mi solución a la práctica de **CIFRADO HILL**, donde implemento HTML y JavaScript para realizar las funciones de encriptado y desencriptado de la palabra de origen.


---

## Código index.html

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Encriptación Hill</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <div class="container">
        <h1>Encriptación Hill</h1>
        
        <div class="section">
            <label for="mensaje">Mensaje (máx. 30 caracteres)</label>
            <textarea id="mensaje" maxlength="30" placeholder="Escribe tu mensaje aquí..."></textarea>
            <div class="char-count">0/30</div>
        </div>

        <div class="section">
            <h3>Matriz del mensaje</h3>
            <div id="matrizMensaje" class="matriz-display"></div>
        </div>

        <div class="section">
            <h3>Matriz clave (2x2)</h3>
            <div class="matriz-input">
                <input type="number" id="k11" placeholder="a">
                <input type="number" id="k12" placeholder="b">
                <input type="number" id="k21" placeholder="c">
                <input type="number" id="k22" placeholder="d">
            </div>
        </div>

        <button id="encriptar">Encriptar</button>
        <button id="desencriptar">Desencriptar</button>

        <div class="section">
            <h3>Resultado</h3>
            <div id="resultado" class="resultado-box"></div>
        </div>
    </div>

    <script src="jscript.js"></script>
</body>
</html>

---

## Código en Jscript.

```
const mensaje = document.getElementById('mensaje');
const charCount = document.querySelector('.char-count');
const matrizMensaje = document.getElementById('matrizMensaje');
const k11 = document.getElementById('k11');
const k12 = document.getElementById('k12');
const k21 = document.getElementById('k21');
const k22 = document.getElementById('k22');
const btnEncriptar = document.getElementById('encriptar');
const btnDesencriptar = document.getElementById('desencriptar');
const resultado = document.getElementById('resultado');

// ============ UTILIDADES MATEMÁTICAS ============

function mod(n, m) {
    return ((n % m) + m) % m;
}

function det2x2(mat) {
    return mod(mat[0][0] * mat[1][1] - mat[0][1] * mat[1][0], 26);
}

function inversoMod26(a) {
    a = mod(a, 26);
    for (let i = 1; i < 26; i++) {
        if (mod(a * i, 26) === 1) return i;
    }
    return null;
}

function matrizInversa(key) {
    const det = det2x2(key);
    const detInv = inversoMod26(det);
    if (detInv === null) return null;

    const a = key[0][0], b = key[0][1];
    const c = key[1][0], d = key[1][1];

    // Inversa 2x2: (1/det) * [[d, -b], [-c, a]] mod 26
    return [
        [mod(detInv * d, 26), mod(detInv * -b, 26)],
        [mod(detInv * -c, 26), mod(detInv * a, 26)]
    ];
}

// Hace que la clave SIEMPRE sea invertible módulo 26
function hacerClaveInvertible(key) {
    // Normalizar elementos a 0..25
    let a = mod(key[0][0], 26);
    let b = mod(key[0][1], 26);
    let c = mod(key[1][0], 26);
    let d = mod(key[1][1], 26);

    let base = [
        [a, b],
        [c, d]
    ];

    if (inversoMod26(det2x2(base)) !== null) {
        return base;
    }

    // Probar ajustar solo 'd'
    for (let delta = 1; delta < 26; delta++) {
        let d2 = mod(d + delta, 26);
        let k = [
            [a, b],
            [c, d2]
        ];
        if (inversoMod26(det2x2(k)) !== null) {
            return k;
        }
    }

    // Probar ajustar 'c'
    for (let delta = 1; delta < 26; delta++) {
        let c2 = mod(c + delta, 26);
        let k = [
            [a, b],
            [c2, d]
        ];
        if (inversoMod26(det2x2(k)) !== null) {
            return k;
        }
    }

    // Probar ajustar 'b'
    for (let delta = 1; delta < 26; delta++) {
        let b2 = mod(b + delta, 26);
        let k = [
            [a, b2],
            [c, d]
        ];
        if (inversoMod26(det2x2(k)) !== null) {
            return k;
        }
    }

    // Probar ajustar 'a'
    for (let delta = 1; delta < 26; delta++) {
        let a2 = mod(a + delta, 26);
        let k = [
            [a2, b],
            [c, d]
        ];
        if (inversoMod26(det2x2(k)) !== null) {
            return k;
        }
    }

    // Si nada funcionó (caso ultra raro), usar una clave fija válida
    return [
        [3, 3],
        [2, 5]
    ];
}

// ============ LÓGICA DE INTERFAZ ============

// Actualizar contador de caracteres
mensaje.addEventListener('input', () => {
    const len = mensaje.value.length;
    charCount.textContent = `${len}/30`;
    mostrarMatrizMensaje();
});

// Mostrar matriz del mensaje (solo letras, sin espacios)
function mostrarMatrizMensaje() {
    const texto = mensaje.value.toUpperCase().replace(/[^A-Z]/g, '');
    
    if (texto.length === 0) {
        matrizMensaje.textContent = 'Escribe un mensaje primero...';
        return;
    }
    
    const valores = texto.split('').map(char => char.charCodeAt(0) - 65);
    
    // Agrupar en pares
    let matriz = '[';
    for (let i = 0; i < valores.length; i += 2) {
        if (i > 0) matriz += ' ';
        const v1 = valores[i];
        const v2 = (i + 1 < valores.length) ? valores[i + 1] : 23; // padding con 'X'
        matriz += `[${v1}, ${v2}]`;
    }
    matriz += ']';
    
    matrizMensaje.textContent = matriz;
}

// ============ ENCRIPTAR (respetando espacios) ============

btnEncriptar.addEventListener('click', () => {
    resultado.classList.remove('error');

    let key = [
        [parseInt(k11.value) || 0, parseInt(k12.value) || 0],
        [parseInt(k21.value) || 0, parseInt(k22.value) || 0]
    ];

    // Convertimos la clave en una clave siempre válida
    key = hacerClaveInvertible(key);
    
    const textoOriginal = mensaje.value.toUpperCase();
    const soloLetras = textoOriginal.replace(/[^A-Z]/g, '');
    
    if (soloLetras.length === 0) {
        resultado.textContent = 'Error: Ingresa un mensaje';
        resultado.classList.add('error');
        return;
    }
    
    // Convertir solo letras a números
    let numeros = soloLetras.split('').map(char => char.charCodeAt(0) - 65);
    
    // Agregar padding si es impar (para la operación de Hill)
    if (numeros.length % 2 !== 0) {
        numeros.push(23); // 'X'
    }
    
    // Encriptar únicamente las letras
    let encriptadoSoloLetras = '';
    for (let i = 0; i < numeros.length; i += 2) {
        const v1 = numeros[i];
        const v2 = numeros[i + 1];
        
        const c1 = mod(key[0][0] * v1 + key[0][1] * v2, 26);
        const c2 = mod(key[1][0] * v1 + key[1][1] * v2, 26);
        
        encriptadoSoloLetras += String.fromCharCode(65 + c1);
        encriptadoSoloLetras += String.fromCharCode(65 + c2);
    }

    // Reconstruir texto cifrado respetando espacios y caracteres no letras
    let idx = 0;
    let encriptadoConEspacios = '';
    for (let ch of textoOriginal) {
        const isLetter = ch.toUpperCase() >= 'A' && ch.toUpperCase() <= 'Z';
        if (isLetter) {
            encriptadoConEspacios += encriptadoSoloLetras[idx++] || '';
        } else {
            encriptadoConEspacios += ch; // deja espacios, comas, etc.
        }
    }

    // Si sobran letras cifradas (por el padding), agrégalas al final
    if (idx < encriptadoSoloLetras.length) {
        encriptadoConEspacios += encriptadoSoloLetras.slice(idx);
    }
    
    resultado.textContent = encriptadoConEspacios;
});

// ============ DESENCRIPTAR (respetando espacios) ============

btnDesencriptar.addEventListener('click', () => {
    resultado.classList.remove('error');

    let key = [
        [parseInt(k11.value) || 0, parseInt(k12.value) || 0],
        [parseInt(k21.value) || 0, parseInt(k22.value) || 0]
    ];

    // Usamos la misma lógica: corregir clave a una invertible
    key = hacerClaveInvertible(key);

    const textoCifradoConEspacios = resultado.textContent.toUpperCase();
    const soloLetrasCifradas = textoCifradoConEspacios.replace(/[^A-Z]/g, '');

    if (soloLetrasCifradas.length === 0) {
        resultado.textContent = 'Error: No hay texto cifrado en el resultado';
        resultado.classList.add('error');
        return;
    }

    const invKey = matrizInversa(key);

    if (!invKey) {
        resultado.textContent = 'Error inesperado con la clave generada';
        resultado.classList.add('error');
        return;
    }

    // Desencriptar solo las letras (de 2 en 2)
    let numeros = soloLetrasCifradas.split('').map(c => c.charCodeAt(0) - 65);
    let textoPlanoSoloLetras = '';

    for (let i = 0; i < numeros.length; i += 2) {
        if (i + 1 >= numeros.length) break; // si queda una sola, se ignora (era padding)
        const c1 = numeros[i];
        const c2 = numeros[i + 1];

        const p1 = mod(invKey[0][0] * c1 + invKey[0][1] * c2, 26);
        const p2 = mod(invKey[1][0] * c1 + invKey[1][1] * c2, 26);

        textoPlanoSoloLetras += String.fromCharCode(65 + p1);
        textoPlanoSoloLetras += String.fromCharCode(65 + p2);
    }

    // Reconstruir texto plano respetando espacios y otros caracteres
    let idx = 0;
    let textoPlanoConEspacios = '';
    for (let ch of textoCifradoConEspacios) {
        const isLetter = ch.toUpperCase() >= 'A' && ch.toUpperCase() <= 'Z';
        if (isLetter && idx < textoPlanoSoloLetras.length) {
            textoPlanoConEspacios += textoPlanoSoloLetras[idx++] || '';
        } else {
            textoPlanoConEspacios += ch; // conserva espacios, comas, etc.
        }
    }

    // Quitar posible padding 'X' al final (si quedó visible)
    if (textoPlanoConEspacios.endsWith('X')) {
        textoPlanoConEspacios = textoPlanoConEspacios.slice(0, -1);
    }

    resultado.textContent = textoPlanoConEspacios;
});

```


## Código de estilos en css.

```

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Courier New', Courier, monospace;
    background: linear-gradient(135deg, #125259 90%, #0D0D0D 1%);
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
}

.container {
    background: #0D0D0D;
    max-width: 600px;
    width: 100%;
    padding: 40px;
    border-radius: 8px;
    box-shadow: 0 10px 40px #F21F0C;
}

h1 {
    color: #69B7BF;
    text-align: center;
    margin-bottom: 30px;
    font-size: 32px;
    font-weight: 600;
}

h3 {
    color: #69B7BF;
    margin-bottom: 15px;
    font-size: 18px;
    font-weight: 500;
}

.section {
    margin-bottom: 25px;
}

label {
    display: block;
    color: #F2600C;
    margin-bottom: 8px;
    font-weight: 500;
}

textarea {
    width: 100%;
    padding: 12px;
    border: 2px solid #e0e0e0;
    border-radius: 4px;
    font-size: 16px;
    resize: vertical;
    min-height: 80px;
    font-family: inherit;
    transition: border-color 0.3s;
}

textarea:focus {
    outline: none;
    border-color: #125259;
}

.char-count {
    text-align: right;
    color: #666;
    font-size: 12px;
    margin-top: 5px;
}

.matriz-display {
    background: #f5f5f5;
    padding: 20px;
    border-radius: 4px;
    border: 2px solid #e0e0e0;
    min-height: 60px;
    font-family: 'Courier New', monospace;
    color: #333;
    line-height: 1.8;
}

.matriz-input {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    max-width: 200px;
}

.matriz-input input {
    padding: 12px;
    border: 2px solid #e0e0e0;
    border-radius: 4px;
    font-size: 18px;
    text-align: center;
    transition: border-color 0.3s;
}

.matriz-input input:focus {
    outline: none;
    border-color: #125259;
}

button {
    width: 100%;
    padding: 15px;
    background: #69B7BF;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 18px;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.3s;
    margin: 20px 0;
}

button:hover {
    background: #125259;
}

button:active {
    transform: scale(0.98);
}

.resultado-box {
    background: #f5f5f5;
    padding: 20px;
    border-radius: 4px;
    border: 2px solid #e0e0e0;
    min-height: 60px;
    font-family: 'Courier New', monospace;
    color: #333;
    word-wrap: break-word;
}

.error {
    color: #F21F0C;
    background: #ffebee;
    border-color: #F21F0C;
}

```

### Clonar el repositorio
```bash
git clone https://github.com/ManuelTelloM/equisde.git
cd equisde
```

## Estructura básica de la página.
```
En esta seccion llamda index, se tomo en consideracion la estructura básica de la página web, como por ejemplo el tamaño del liezo, los bordes, botones, ancho y alto de cada elemento, se asignarion los layouts, cuadros de texto, etc.

```

## Funcionamiento de la página.
```
Para esta seccion ya se programo el funcionamiento como tal de la pagina, que es lo que va a hacer cada layout, las funciones de encriptado y la funcion inversa para poder realizar de la menera más optima el desencriptado del mensaje, ademas de que tambien se añade las acciones de los botones para encriptar el mensaje y su contraparte el desencriptado.

```

## Diseño de la página.
```
Aqui se le cambio el color de los elementos de la pagiona, ademas de que se le aplico oto tipo de tipografia a la misma y se le coloco una paleta de colores a la pagina que la dejo bastante llamtiva visualmente e intuitiva.

```
 ## hola.
 
 Hola :3 esto es un mensaje.

 ---
 
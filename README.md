<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Horario Académico Personalizado</title>
    <style>
        :root {
            --bg: #f8fafc;
            --primary: #2563eb;
            --text: #1e293b;
        }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: var(--bg); color: var(--text); padding: 20px; }
        .container { max-width: 1200px; margin: auto; background: white; padding: 25px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); }
        h1 { text-align: center; color: var(--primary); margin-bottom: 30px; }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 15px; }
        .card { border: 2px solid #e2e8f0; border-radius: 12px; padding: 15px; transition: 0.3s; }
        .card:hover { border-color: var(--primary); }
        .card b { display: block; margin-bottom: 10px; font-size: 0.95rem; height: 2.5em; overflow: hidden; }
        label { margin-right: 12px; font-weight: 500; cursor: pointer; }
        button { margin-top: 25px; padding: 15px; width: 100%; background: var(--primary); color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; font-size: 1.1rem; }
        
        table { width: 100%; border-collapse: collapse; margin-top: 20px; table-layout: fixed; }
        th, td { border: 1px solid #e2e8f0; padding: 0; text-align: center; height: 45px; }
        th { background: var(--primary); color: white; font-size: 14px; padding: 10px 0; }
        .hora-col { width: 120px; background: #f1f5f9; font-weight: bold; font-size: 11px; }

        /* Estilo de los bloques de curso */
        .course-block {
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10px;
            font-weight: bold;
            padding: 0 5px;
            color: rgba(0,0,0,0.7);
            border-bottom: 1px solid rgba(255,255,255,0.3); /* Unión visual */
        }
        .receso { background: #cbd5e1; font-size: 9px; color: #64748b; }
    </style>
</head>
<body>

<div class="container" id="selection">
    <h1>Selecciona tus Grupos</h1>
    <div class="grid" id="cursosContainer"></div>
    <button id="btn">Generar Mi Horario</button>
</div>

<div class="container" id="schedule" style="display:none;">
    <h1>Mi Horario Semanal</h1>
    <div style="overflow-x:auto;">
        <table>
            <thead>
                <tr>
                    <th class="hora-col">Hora</th>
                    <th>Lunes</th><th>Martes</th><th>Miércoles</th><th>Jueves</th><th>Viernes</th>
                </tr>
            </thead>
            <tbody id="body"></tbody>
        </table>
    </div>
    <button onclick="location.reload()" style="background:#64748b;">Atrás / Editar</button>
</div>

<script>
const cursosInfo = [
    { n: "Taller de diseño arquitectónico 1", color: "#bfdbfe" },
    { n: "Representación arquitectónica 1", color: "#bbf7d0" },
    { n: "Introducción a la Teoría e Historia de la arquitectura", color: "#fef08a" },
    { n: "Apreciación del arte universal", color: "#fed7aa" },
    { n: "Metodología del trabajo académico", color: "#ddd6fe" },
    { n: "Fotografía y cine", color: "#fbcfe8" },
    { n: "Inglés", color: "#99f6e4" }
];

const container = document.getElementById("cursosContainer");
cursosInfo.forEach((c, i) => {
    let div = document.createElement("div");
    div.className = "card";
    div.innerHTML = `<b>${c.n}</b>
    ${['A','B','C','D'].map(g => `<label><input type="radio" name="c${i}" value="${g}"> ${g}</label>`).join('')}`;
    container.appendChild(div);
});

const hours = [
    "07:00 - 07:50", "07:50 - 08:40", "RECESO",
    "08:50 - 09:40", "09:40 - 10:30", "RECESO",
    "10:40 - 11:30", "11:30 - 12:20", "12:20 - 13:10", "13:10 - 14:00",
    "RECESO",
    "14:00 - 14:50", "14:50 - 15:40", "RECESO",
    "15:50 - 16:40", "16:40 - 17:30", "RECESO",
    "17:40 - 18:30", "18:30 - 19:20", "19:20 - 20:10", "20:10 - 21:00"
];

const days = ["Lunes", "Martes", "Miércoles", "Jueves", "Viernes"];

function generarHorario() {
    let seleccion = {};
    cursosInfo.forEach((c, i) => {
        let sel = document.querySelector(`input[name="c${i}"]:checked`);
        if (sel) seleccion[c.n] = { grupo: sel.value, color: c.color };
    });

    if (Object.keys(seleccion).length === 0) { alert("Elige al menos un curso"); return; }

    let data = {};
    hours.forEach(h => { data[h] = {}; days.forEach(d => data[h][d] = null); });

    function asig(dia, listaH, curso, info) {
        listaH.forEach(h => { if(data[h]) data[h][dia] = { nombre: curso, color: info.color, g: info.grupo }; });
    }

    for (let c in seleccion) {
        const info = seleccion[c];
        const g = info.grupo;
        const esMati = (g === "A" || g === "C");

        if (c.includes("Taller")) {
            const h = esMati ? ["08:50 - 09:40","09:40 - 10:30","10:40 - 11:30","11:30 - 12:20","12:20 - 13:10"] 
                             : ["15:50 - 16:40","16:40 - 17:30","17:40 - 18:30","18:30 - 19:20","19:20 - 20:10"];
            asig("Lunes", h, c, info); asig("Jueves", h, c, info);
        }
        if (c.includes("Representación")) {
            const h = esMati ? ["07:00 - 07:50","07:50 - 08:40","08:50 - 09:40","09:40 - 10:30"] 
                             : ["14:00 - 14:50","14:50 - 15:40","15:50 - 16:40","16:40 - 17:30"];
            asig("Martes", h, c, info);
        }
        if (c.includes("Historia")) {
            const h = esMati ? ["10:40 - 11:30","11:30 - 12:20","12:20 - 13:10","13:10 - 14:00"] 
                             : ["17:40 - 18:30","18:30 - 19:20","19:20 - 20:10","20:10 - 21:00"];
            asig("Viernes", h, c, info);
        }
        if (c.includes("apreciación")) {
            const h = esMati ? ["10:40 - 11:30","11:30 - 12:20","12:20 - 13:10","13:10 - 14:00"] 
                             : ["17:40 - 18:30","18:30 - 19:20","19:20 - 20:10","20:10 - 21:00"];
            asig("Miércoles", h, c, info);
        }
        if (c.includes("Metodología")) {
            const h = esMati ? ["08:50 - 09:40","09:40 - 10:30"] : ["15:50 - 16:40","16:40 - 17:30"];
            asig("Miércoles", h, c, info); asig("Viernes", h, c, info);
        }
        if (c.includes("Fotografía")) {
            const h = esMati ? ["07:00 - 07:50","07:50 - 08:40"] : ["14:00 - 14:50","14:50 - 15:40"];
            asig("Miércoles", h, c, info); asig("Viernes", h, c, info);
        }
        if (c.includes("Inglés")) {
            const h = esMati ? ["10:40 - 11:30","11:30 - 12:20","12:20 - 13:10"] 
                             : ["17:40 - 18:30","18:30 - 19:20","19:20 - 20:10"];
            asig("Martes", h, c, info);
        }
    }

    const body = document.getElementById("body");
    body.innerHTML = "";

    hours.forEach(h => {
        let row = `<tr><td class="hora-col">${h}</td>`;
        days.forEach(d => {
            if (h === "RECESO") {
                row += `<td class="receso">--</td>`;
            } else {
                let cell = data[h][d];
                if (cell) {
                    row += `<td style="background:${cell.color}"><div class="course-block">${cell.nombre}</div></td>`;
                } else {
                    row += `<td></td>`;
                }
            }
        });
        row += "</tr>";
        body.innerHTML += row;
    });

    document.getElementById("selection").style.display = "none";
    document.getElementById("schedule").style.display = "block";
}

document.getElementById("btn").addEventListener("click", generarHorario);
</script>

</body>
</html>

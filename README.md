<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>PhysioTest</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f2f2f2;
    }
    h1, h2 {
      text-align: center;
      color: #2c3e50;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      padding: 10px;
      border: 1px solid #ccc;
      text-align: center;
    }
    input[type="number"], input[type="text"] {
      width: 80px;
      padding: 5px;
      text-align: center;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
      display: block;
      margin-left: auto;
      margin-right: auto;
      background-color: #3498db;
      color: white;
      border: none;
      border-radius: 6px;
    }
    #resumoFisio {
      white-space: pre-wrap;
      background: #fff;
      padding: 15px;
      margin-top: 20px;
      border-radius: 8px;
      border: 1px solid #ccc;
    }

    /* Estilo responsivo para dispositivos móveis */
    @media screen and (max-width: 768px) {
      body {
        font-size: 16px;
        padding: 10px;
      }
      h1, h2, h3 {
        font-size: 20px !important;
        text-align: center;
      }
      table {
        width: 100%;
        display: block;
        overflow-x: auto;
      }
      input[type="number"], input[type="text"], textarea {
        width: 100% !important;
        margin: 5px 0;
        font-size: 16px;
      }
      button {
        width: 100% !important;
        font-size: 18px;
        padding: 10px;
        margin-bottom: 10px;
      }
      .container, .content, .card {
        width: 100% !important;
        padding: 5px !important;
      }
      .table-responsive {
        overflow-x: auto;
        display: block;
        width: 100%;
      }
    }
  </style>
</head>
<body>

  <h1>PhysioTest</h1>
  <h2>Avaliação Fisioterapêutica</h2>

  <table>
    <thead>
      <tr>
        <th>Movimento</th>
        <th>Referência</th>
        <th>Lado D (°)</th>
        <th>Lado E (°)</th>
        <th>% D</th>
        <th>% E</th>
        <th>Interpretação D</th>
        <th>Interpretação E</th>
      </tr>
    </thead>
    <tbody id="adm-tabela-body">
    </tbody>
  </table>

  <button onclick="gerarResumoFisio()">Gerar Resumo Fisioterapêutico</button>

  <div id="resumoFisio"></div>

  <script>
    const movimentos = [
      { nome: "Flexão de Quadril", ref: 130 },
      { nome: "Extensão de Quadril", ref: 20 },
      { nome: "Abdução de Quadril", ref: 50 },
      { nome: "Adução de Quadril", ref: 30 },
      { nome: "Flexão de Joelho", ref: 140 },
      { nome: "Extensão de Joelho", ref: 0 },
      { nome: "Dorsiflexão do Tornozelo", ref: 30 },
      { nome: "Flexão Plantar", ref: 50 },
      { nome: "Flexão de Ombro", ref: 180 },
      { nome: "Abdução de Ombro", ref: 180 },
      { nome: "Flexão de Cotovelo", ref: 150 },
      { nome: "Extensão de Cotovelo", ref: 0 },
      { nome: "Flexão de Punho", ref: 90 },
      { nome: "Extensão de Punho", ref: 70 }
    ];

    function avaliarPercentualADM(valor, ref) {
      if (ref === 0) {
        if (valor <= 1) return { pct: "100%", cor: "green", texto: "ADM preservada" };
        if (valor <= 10) return { pct: "-", cor: "orange", texto: "ADM moderadamente reduzida" };
        return { pct: "-", cor: "red", texto: "ADM comprometida" };
      }
      const pct = Math.round((valor / ref) * 100);
      if (pct >= 90) return { pct: pct + "%", cor: "green", texto: "ADM preservada" };
      if (pct >= 70) return { pct: pct + "%", cor: "orange", texto: "ADM moderadamente reduzida" };
      return { pct: pct + "%", cor: "red", texto: "ADM comprometida" };
    }

    function atualizarInterpretaçãoADM(i) {
      const ref = movimentos[i].ref;
      ["D", "E"].forEach((lado) => {
        const input = document.getElementById(`adm${lado}${i}`);
        const pctCell = document.getElementById(`pct${lado}${i}`);
        const interp = document.getElementById(`interp${lado}${i}`);
        const val = parseFloat(input.value);
        if (isNaN(val)) {
          pctCell.textContent = "";
          interp.textContent = "";
          interp.style.color = "#333";
        } else {
          const res = avaliarPercentualADM(val, ref);
          pctCell.textContent = res.pct;
          interp.textContent = res.texto;
          interp.style.color = res.cor;
        }
      });
    }

    function gerarResumoFisio() {
      let resumo = "";
      movimentos.forEach((mov, i) => {
        const d = parseFloat(document.getElementById("admD" + i)?.value);
        const e = parseFloat(document.getElementById("admE" + i)?.value);
        const ref = mov.ref;

        const avaliar = (val) => {
          if (isNaN(val)) return null;
          if (ref === 0) {
            if (val === 0) return null;
            if (val <= 10) return "⚠️ ADM moderadamente reduzida em " + mov.nome;
            return "❌ ADM comprometida em " + mov.nome;
          } else {
            const pct = (val / ref) * 100;
            if (pct >= 90) return null;
            if (pct >= 70) return "⚠️ ADM moderadamente reduzida em " + mov.nome;
            return "❌ ADM comprometida em " + mov.nome;
          }
        };

        const rD = avaliar(d);
        const rE = avaliar(e);
        if (rD) resumo += rD + " à direita (" + d + "° de " + ref + "°).\n";
        if (rE) resumo += rE + " à esquerda (" + e + "° de " + ref + "°).\n";
      });

      if (!resumo) resumo = "✅ Nenhum déficit encontrado nas avaliações.";
      document.getElementById("resumoFisio").textContent = resumo;
    }

    document.addEventListener("DOMContentLoaded", () => {
      const corpo = document.getElementById("adm-tabela-body");
      movimentos.forEach((m, i) => {
        const linha = document.createElement("tr");
        linha.innerHTML = `
          <td>${m.nome}</td>
          <td>0–${m.ref}°</td>
          <td><input type="number" id="admD${i}" onchange="atualizarInterpretaçãoADM(${i})" /></td>
          <td><input type="number" id="admE${i}" onchange="atualizarInterpretaçãoADM(${i})" /></td>
          <td id="pctD${i}"></td>
          <td id="pctE${i}"></td>
          <td id="interpD${i}"></td>
          <td id="interpE${i}"></td>
        `;
        corpo.appendChild(linha);
      });
    });
  </script>

</body>
</html>

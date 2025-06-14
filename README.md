<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Meu Jogo na Mega-Sena</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      background: #f9f9f9;
    }
    h1 {
      color: #2d2d2d;
      text-align: center;
    }
    .meus-numeros-container {
      text-align: center;
      margin-bottom: 20px;
    }
    .numeros, .dezenas {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
      justify-content: center;
      margin-top: 8px;
    }
    .numero {
      width: 40px;
      height: 40px;
      background: #ddd;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      font-size: 16px;
    }
    .acertou {
      background: #4caf50;
      color: white;
    }
    .container-principal {
      display: flex;
      gap: 20px;
      align-items: flex-start;
      flex-wrap: wrap;
    }
    #historicoSorteios {
      flex: 1 1 400px;
      max-width: 600px;
    }
    #grafico-container {
      flex: 1 1 300px;
      max-width: 500px;
    }
    .sorteio-info {
      background: white;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 8px;
      box-shadow: 0 0 5px #ccc;
    }
    .titulo-sorteio {
      margin-bottom: 6px;
      font-size: 18px;
      font-weight: bold;
    }
    .premio {
      margin-top: 10px;
      font-size: 14px;
    }
    .detalhes-toggle {
      margin-top: 10px;
      cursor: pointer;
      color: #0077cc;
      text-decoration: underline;
    }
    .detalhes {
      display: none;
      margin-top: 10px;
      font-size: 14px;
      color: #444;
    }

    @media (max-width: 850px) {
      .container-principal {
        flex-direction: column;
      }
      #historicoSorteios, #grafico-container {
        max-width: 100%;
        flex-basis: auto;
      }
    }
  </style>
</head>
<body>

  <h1>Resultados da Mega-Sena</h1>

  <div class="meus-numeros-container">
    <strong>Seus números:</strong>
    <div class="numeros" id="meusNumeros"></div>
  </div>

  <hr />

  <div class="container-principal">
    <div id="historicoSorteios"></div>

    <div id="grafico-container">
      <h2>Seus acertos nos últimos concursos</h2>
      <canvas id="graficoAcertos"></canvas>
    </div>
  </div>

  <script>
    const meusNumeros = [4, 7, 25, 39, 44, 53];
    const acertosPorConcurso = [];

    const faixaMap = {
      1: 'Sena',
      2: 'Quina',
      3: 'Quadra'
    };

    function mostrarMeusNumeros() {
      const c = document.getElementById('meusNumeros');
      c.innerHTML = '';
      meusNumeros.forEach(n => {
        const d = document.createElement('div');
        d.classList.add('numero');
        d.textContent = n;
        c.appendChild(d);
      });
    }

    function formatarValor(valor) {
      return parseFloat(valor).toLocaleString('pt-BR', {minimumFractionDigits: 2});
    }

    function estimarCDB(valor) {
      const taxaMensal = 0.0079;
      return valor * taxaMensal;
    }

    function mostrarSorteio(s) {
      const c = document.getElementById('historicoSorteios');
      const div = document.createElement('div');
      div.classList.add('sorteio-info');

      const acertos = s.listaDezenas.filter(num => meusNumeros.includes(Number(num)));
      acertosPorConcurso.push({ concurso: s.numero, acertos: acertos.length });

      div.innerHTML = `
        <div class="titulo-sorteio">Concurso ${s.numero} - ${s.dataApuracao}</div>
        <p><strong>Dezenas sorteadas:</strong></p>
      `;

      const dezenasDiv = document.createElement('div');
      dezenasDiv.classList.add('dezenas');
      s.listaDezenas.forEach(num => {
        const nd = document.createElement('div');
        nd.classList.add('numero');
        if (meusNumeros.includes(Number(num))) nd.classList.add('acertou');
        nd.textContent = num;
        dezenasDiv.appendChild(nd);
      });

      div.appendChild(dezenasDiv);

      let premiacaoHtml = '';
      if (s.listaRateioPremio) {
        premiacaoHtml += `<div class="premio"><strong>Premiação:</strong><ul>`;
        s.listaRateioPremio.forEach(p => {
          const ganhadores = p.quantidadeGanhadores ?? p.numeroDeGanhadores ?? 'Aguardando apuração';
          const faixa = faixaMap[p.faixa] || p.faixa;
          premiacaoHtml += `<li>${faixa}: ${ganhadores} ganhador(es) - R$ ${formatarValor(p.valorPremio)}</li>`;
        });
        premiacaoHtml += `</ul></div>`;
      }

      premiacaoHtml += `<div class="premio"><strong>Você acertou:</strong> ${acertos.length} número(s)</div>`;

      premiacaoHtml += `
        <div class="detalhes-toggle" onclick="this.nextElementSibling.style.display = this.nextElementSibling.style.display === 'none' ? 'block' : 'none'">Ver detalhes</div>
        <div class="detalhes">
          ${s.acumulado ? '<p><strong>Acumulado!</strong></p>' : ''}
          <p>Prêmio estimado próximo concurso: R$ ${formatarValor(s.valorEstimadoProximoConcurso || 0)}</p>
          <p>Rendimento estimado/mês (CDB 105% CDI): R$ ${formatarValor(estimarCDB(s.valorEstimadoProximoConcurso || 0))}</p>
        </div>
      `;

      div.innerHTML += premiacaoHtml;
      c.appendChild(div);
    }

    function exibirGrafico() {
      const ctx = document.getElementById('graficoAcertos').getContext('2d');
      const labels = acertosPorConcurso.map(item => `Conc. ${item.concurso}`);
      const dados = acertosPorConcurso.map(item => item.acertos);

      new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [{
            label: 'Acertos por concurso',
            data: dados,
            borderColor: '#4caf50',
            backgroundColor: 'rgba(76, 175, 80, 0.2)',
            tension: 0.3,
            fill: true,
            pointRadius: 5
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true,
              stepSize: 1,
              max: 6
            }
          }
        }
      });
    }

    async function carregarUltimos5Sorteios() {
      try {
        const concursos = [];
        const urlBase = 'https://servicebus2.caixa.gov.br/portaldeloterias/api/megasena/';
        const proxyBase = 'https://api.allorigins.win/raw?url=';

        const res = await fetch(proxyBase + encodeURIComponent(urlBase));
        const atual = await res.json();
        concursos.push(atual);

        for (let i = 1; i <= 4; i++) {
          const numeroConcurso = atual.numero - i;
          const resAnt = await fetch(proxyBase + encodeURIComponent(urlBase + numeroConcurso));
          const dataAnt = await resAnt.json();
          concursos.push(dataAnt);
        }

        concursos.forEach(s => mostrarSorteio(s));
        exibirGrafico();
      } catch (e) {
        console.error('Erro ao carregar dados:', e);
        document.getElementById('historicoSorteios').innerHTML =
          '<p>Erro ao carregar dados dos sorteios. Tente recarregar a página.</p>';
      }
    }

    mostrarMeusNumeros();
    carregarUltimos5Sorteios();
  </script>

</body>
</html>

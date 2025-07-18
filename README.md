<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Gerenciamento de Máquinas - Patrimônio</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #121212;
      color: white;
      margin: 0;
      padding: 20px;
    }
    h1 {
      text-align: center;
      margin-bottom: 20px;
    }
    input, select, button {
      padding: 8px;
      margin: 5px;
      border-radius: 5px;
      border: none;
      font-size: 14px;
    }
    input, select {
      background-color: #1e1e1e;
      color: white;
      border: 1px solid #333;
      min-width: 150px;
    }
    button {
      background-color: #03dac5;
      color: black;
      cursor: pointer;
      font-weight: bold;
    }
    button:hover {
      background-color: #029e96;
      color: white;
    }
    #filtro {
      margin: 10px 0;
      padding: 8px;
      width: 100%;
      max-width: 400px;
      border-radius: 5px;
      border: 1px solid #333;
      background-color: #1e1e1e;
      color: white;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
      background-color: #1e1e1e;
      border-radius: 5px;
      overflow: hidden;
    }
    th, td {
      padding: 10px;
      border-bottom: 1px solid #333;
      text-align: center;
    }
    th {
      background-color: #03dac5;
      color: black;
    }
    tr.estourado {
      background-color: yellow !important;
      color: black !important;
    }
    td button {
      background: none;
      border: none;
      color: #f44336;
      cursor: pointer;
      font-size: 16px;
    }
    td button:hover {
      color: #ff7961;
    }
    td input[type="checkbox"] {
      transform: scale(1.2);
      cursor: pointer;
    }
    td input[type="text"].observacao {
      background-color: #1e1e1e;
      color: white;
      border: 1px solid #333;
      border-radius: 3px;
      padding: 4px 6px;
      width: 140px;
      font-size: 14px;
    }
  </style>
</head>
<body>

  <h1>Controle de Máquinas no Armário</h1>

  <div>
    <input type="text" id="nomeCompleto" placeholder="Nome Completo" />
    <input type="text" id="nome" placeholder="Nome Patrimônio (ex: BRMH001)" oninput="validarNome()" />
    <select id="tipo">
      <option value="Desktop">Desktop</option>
      <option value="Notebook">Notebook</option>
    </select>
    <input type="text" id="ipn" placeholder="IPN" />
    <input type="text" id="registro" placeholder="Nº Registro" />
    <button onclick="adicionarRegistro()">Registrar</button>
  </div>

  <input type="text" id="filtro" placeholder="🔍 Pesquisar registros..." oninput="filtrarTabela()" />

  <table>
    <thead>
      <tr>
        <th>Nome Completo</th>
        <th>Nome Patrimônio</th>
        <th>IPN</th>
        <th>Tipo</th>
        <th>Registro</th>
        <th>Data</th>
        <th>Ação</th>
        <th>Saída</th>
        <th>Excluir</th>
        <th>Observação</th>
      </tr>
    </thead>
    <tbody id="tabelaRegistros"></tbody>
  </table>

  <script>
    let registros = [];

    function validarNome() {
      const nomeInput = document.getElementById('nome');
      const tipo = document.getElementById('tipo').value;
      const valor = nomeInput.value.toUpperCase();
      const prefixos = tipo === 'Desktop' ? ['BRD'] : ['BRMH', 'BRLH', 'BRXH'];

      if (!prefixos.some(p => valor.startsWith(p))) {
        nomeInput.style.border = '2px solid red';
      } else {
        nomeInput.style.border = '';
      }
    }

    function adicionarRegistro() {
      const nomeCompleto = document.getElementById('nomeCompleto').value.trim();
      const nome = document.getElementById('nome').value.trim().toUpperCase();
      const tipo = document.getElementById('tipo').value;
      const ipn = document.getElementById('ipn').value.trim();
      const registro = document.getElementById('registro').value.trim();
      const data = new Date().toISOString();
      const acao = 'Entrada';

      if (!nomeCompleto || !nome || !ipn || !registro) {
        alert('Preencha todos os campos.');
        return;
      }

      const prefixos = tipo === 'Desktop' ? ['BRD'] : ['BRMH', 'BRLH', 'BRXH'];
      if (!prefixos.some(p => nome.startsWith(p))) {
        alert(`Nome inválido para tipo ${tipo}.`);
        return;
      }

      registros.push({ nomeCompleto, nome, ipn, tipo, registro, data, acao, observacao: '' });
      atualizarTabela();
      limparCampos();
    }

    function limparCampos() {
      document.getElementById('nomeCompleto').value = '';
      document.getElementById('nome').value = '';
      document.getElementById('ipn').value = '';
      document.getElementById('registro').value = '';
      document.getElementById('tipo').value = 'Desktop';
      document.getElementById('nome').style.border = '';
    }

    function atualizarTabela() {
      const tbody = document.getElementById('tabelaRegistros');
      tbody.innerHTML = '';
      const agora = new Date();

      registros.forEach((reg, i) => {
        const dataRegistro = new Date(reg.data);
        const diffDias = (agora - dataRegistro) / (1000 * 60 * 60 * 24);
        let acao = reg.acao;
        let classe = '';

        if (diffDias > 10 && acao !== 'Saída') {
          acao = 'ESTOUROU MAN';
          classe = 'estourado';
        }

        const tr = document.createElement('tr');
        tr.className = classe;
        tr.innerHTML = `
          <td style="color: black;">${reg.nomeCompleto}</td>
          <td style="color: black;">${reg.nome}</td>
          <td style="color: black;">${reg.ipn}</td>
          <td style="color: black;">${reg.tipo}</td>
          <td style="color: black;">${reg.registro}</td>
          <td style="color: black;">${dataRegistro.toLocaleString()}</td>
          <td style="color: black;">${acao}</td>
          <td><input type="checkbox" ${reg.acao === 'Saída' ? 'checked disabled' : ''} onchange="marcarSaida(${i}, this)"></td>
          <td><button onclick="excluirRegistro(${i})">🗑️</button></td>
          <td><input type="text" class="observacao" value="${reg.observacao || ''}" placeholder="Observação" onchange="salvarObservacao(${i}, this.value)" /></td>
        `;
        tbody.appendChild(tr);
      });
    }

    function salvarObservacao(index, valor) {
      registros[index].observacao = valor;
      atualizarTabela();
    }

    function marcarSaida(index, checkbox) {
      if (checkbox.checked) {
        registros[index].acao = 'Saída';
        checkbox.disabled = true;
      } else {
        registros[index].acao = 'Entrada';
      }
      atualizarTabela();
    }

    function excluirRegistro(index) {
      if (confirm('Deseja excluir este registro?')) {
        registros.splice(index, 1);
        atualizarTabela();
      }
    }

    function filtrarTabela() {
      const filtro = document.getElementById('filtro').value.toLowerCase();
      const linhas = document.querySelectorAll('#tabelaRegistros tr');
      linhas.forEach(linha => {
        linha.style.display = linha.innerText.toLowerCase().includes(filtro) ? '' : 'none';
      });
    }

    atualizarTabela();
  </script>
</body>
</html>

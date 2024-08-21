<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro de Alunos</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }

        header {
            background-color: #007bff;
            color: white;
            padding: 10px 0;
            text-align: center;
        }

        header h1 {
            margin: 0;
        }

        #cadastro, #editar {
            background-color: white;
            margin: 20px auto;
            padding: 20px;
            border-radius: 8px;
            width: 80%;
            max-width: 600px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        #cadastro h2, #editar h2 {
            margin-bottom: 20px;
        }

        #cadastro label, #editar label {
            display: block;
            margin-bottom: 5px;
        }

        #cadastro input, #editar input {
            width: 100%;
            padding: 8px;
            margin-bottom: 15px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        #cadastro button, #editar button {
            background-color: #007bff;
            color: white;
            padding: 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            width: 100%;
        }

        #cadastro button:hover, #editar button:hover {
            background-color: #0056b3;
        }

        #alunos {
            margin: 20px auto;
            width: 80%;
            max-width: 600px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }

        table, th, td {
            border: 1px solid #ddd;
        }

        th, td {
            padding: 8px;
            text-align: left;
        }

        th {
            background-color: #007bff;
            color: white;
        }

        .actions {
            display: flex;
            gap: 5px;
        }

        .actions button {
            background-color: #28a745;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }

        .actions button.delete {
            background-color: #dc3545;
        }

        .actions button:hover {
            opacity: 0.8;
        }

        footer {
            background-color: #007bff;
            color: white;
            text-align: center;
            padding: 10px 0;
            position: fixed;
            width: 100%;
            bottom: 0;
            display: none; /* Oculta o rodapé */
        }

        #modal {
            display: none;
            position: fixed;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            align-items: center;
            justify-content: center;
        }

        #modalContent {
            background: white;
            padding: 20px;
            border-radius: 8px;
            width: 80%;
            max-width: 500px;
        }

        #modalContent h2 {
            margin: 0;
        }

        #modal button.close {
            background-color: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <header>
        <h1>Cadastro de Alunos</h1>
    </header>

    <section id="cadastro">
        <h2>Formulário de Cadastro</h2>
        <form id="cadastroForm">
            <label for="nome">Nome Completo:</label>
            <input type="text" id="nome" name="nome" required>

            <label for="nomeTutor">Nome do Tutor:</label>
            <input type="text" id="nomeTutor" name="nomeTutor" required>

            <label for="sala">Sala:</label>
            <input type="text" id="sala" name="sala" required>

            <label for="eletiva">Eletiva:</label>
            <input type="text" id="eletiva" name="eletiva" required>

            <button type="submit">Cadastrar</button>
        </form>
    </section>

    <section id="alunos">
        <h2>Alunos Cadastrados</h2>
        <table id="tabelaAlunos">
            <thead>
                <tr>
                    <th>Nome</th>
                    <th>Nome do Tutor</th>
                    <th>Sala</th>
                    <th>Eletiva</th>
                    <th>Ações</th>
                </tr>
            </thead>
            <tbody>
                <!-- Os dados serão inseridos aqui -->
            </tbody>
        </table>
        <button id="gerarPdf">Gerar PDF</button>
    </section>

    <div id="modal">
        <div id="modalContent">
            <h2>Editar Aluno</h2>
            <form id="editarForm">
                <label for="editNome">Nome Completo:</label>
                <input type="text" id="editNome" name="nome" required>

                <label for="editNomeTutor">Nome do Tutor:</label>
                <input type="text" id="editNomeTutor" name="nomeTutor" required>

                <label for="editSala">Sala:</label>
                <input type="text" id="editSala" name="sala" required>

                <label for="editEletiva">Eletiva:</label>
                <input type="text" id="editEletiva" name="eletiva" required>

                <button type="submit">Salvar</button>
                <button type="button" class="close">Fechar</button>
            </form>
        </div>
    </div>

    <footer>
        <!-- Rodapé removido -->
    </footer>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.13/jspdf.plugin.autotable.min.js"></script>
    <script>
        let editarIndex = -1; // Índice do aluno sendo editado

        // Função para carregar dados do localStorage e atualizar a tabela
        function carregarDados() {
            const dados = JSON.parse(localStorage.getItem('alunos')) || [];
            const tableBody = document.querySelector('#tabelaAlunos tbody');
            tableBody.innerHTML = '';

            dados.forEach((aluno, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${aluno.nome}</td>
                    <td>${aluno.nomeTutor}</td>
                    <td>${aluno.sala}</td>
                    <td>${aluno.eletiva}</td>
                    <td class="actions">
                        <button onclick="editarAluno(${index})">Editar</button>
                        <button class="delete" onclick="excluirAluno(${index})">Excluir</button>
                    </td>
                `;
                tableBody.appendChild(row);
            });
        }

        // Função para salvar dados no localStorage
        function salvarDados() {
            const rows = Array.from(document.querySelectorAll('#tabelaAlunos tbody tr'));
            const dados = rows.map(row => {
                const cells = row.querySelectorAll('td');
                return {
                    nome: cells[0].textContent,
                    nomeTutor: cells[1].textContent,
                    sala: cells[2].textContent,
                    eletiva: cells[3].textContent
                };
            });
            localStorage.setItem('alunos', JSON.stringify(dados));
        }

        // Função para adicionar aluno
        document.getElementById('cadastroForm').addEventListener('submit', function(event) {
            event.preventDefault();

            const nome = document.getElementById('nome').value;
            const nomeTutor = document.getElementById('nomeTutor').value;
            const sala = document.getElementById('sala').value;
            const eletiva = document.getElementById('eletiva').value;

            const tableBody = document.querySelector('#tabelaAlunos tbody');
            const row = document.createElement('tr');
            row.innerHTML = `
                <td>${nome}</td>
                <td>${nomeTutor}</td>
                <td>${sala}</td>
                <td>${eletiva}</td>
                <td class="actions">
                    <button onclick="editarAluno(${tableBody.rows.length})">Editar</button>
                    <button class="delete" onclick="excluirAluno(${tableBody.rows.length})">Excluir</button>
                </td>
            `;
            tableBody.appendChild(row);

            // Salva os dados no localStorage
            salvarDados();

            // Limpa o formulário
            document.getElementById('cadastroForm').reset();
        });

        // Função para excluir aluno
        function excluirAluno(index) {
            const dados = JSON.parse(localStorage.getItem('alunos')) || [];
            dados.splice(index, 1);
            localStorage.setItem('alunos', JSON.stringify(dados));
            carregarDados();
        }

        // Função para abrir o modal de edição
        function editarAluno(index) {
            const dados = JSON.parse(localStorage.getItem('alunos')) || [];
            const aluno = dados[index];

            document.getElementById('editNome').value = aluno.nome;
            document.getElementById('editNomeTutor').value = aluno.nomeTutor;
            document.getElementById('editSala').value = aluno.sala;
            document.getElementById('editEletiva').value = aluno.eletiva;

            editarIndex = index;
            document.getElementById('modal').style.display = 'flex';
        }

        // Função para salvar as alterações no modal
        document.getElementById('editarForm').addEventListener('submit', function(event) {
            event.preventDefault();

            const nome = document.getElementById('editNome').value;
            const nomeTutor = document.getElementById('editNomeTutor').value;
            const sala = document.getElementById('editSala').value;
            const eletiva = document.getElementById('editEletiva').value;

            const dados = JSON.parse(localStorage.getItem('alunos')) || [];
            dados[editarIndex] = { nome, nomeTutor, sala, eletiva };
            localStorage.setItem('alunos', JSON.stringify(dados));

            carregarDados();
            document.getElementById('modal').style.display = 'none';
        });

        // Função para fechar o modal
        document.querySelector('#modal .close').addEventListener('click', function() {
            document.getElementById('modal').style.display = 'none';
        });

        // Inicializa a tabela ao carregar a página
        window.onload = function() {
            carregarDados();
        };

        document.getElementById('gerarPdf').addEventListener('click', function() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            doc.setFontSize(16);
            doc.text('Relatório de Alunos Cadastrados', 10, 10);

            const table = document.getElementById('tabelaAlunos');
            const rows = Array.from(table.querySelectorAll('tr')).map(row => Array.from(row.querySelectorAll('td')).map(cell => cell.textContent));

            doc.autoTable({
                startY: 20,
                head: [['Nome', 'Nome do Tutor', 'Sala', 'Eletiva']],
                body: rows.slice(1), // Skip the header row
            });

            doc.save('relatorio_alunos.pdf');
        });
    </script>
</body>
</html>

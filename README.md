<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reserva de Raias</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 20px; }
        .raias { display: inline-flex; flex-wrap: wrap; justify-content: center; gap: 3px; margin: 20px 10px; border: 3px solid black; padding: 0px; border-radius: 5px; }
        .raia { padding: 15px 20px; border-left: 5px solid red; border-right: 5px solid red; border-radius: 5px; cursor: pointer; background: #007BFF; min-width: 80px; text-align: center;}
        .raia.selected { background-color: #8eb1d7; color: white; }
        .raias.vertical { flex-direction: column; width: 350px;}
        .horarios { margin: 20px 0; }
        button { padding: 10px 20px; margin: 5px; cursor: pointer; }
        .horarios button.selected { background-color: #28a745; color: white; }
        #resumo { margin-top: 20px; font-weight: bold; }
        #reservas-lista { margin-top: 20px; text-align: left; display: inline-block; }
    </style>
</head>
<body>
    <h1>Reserva de Raias da Piscina</h1>

    <label for="usuario">Seu nome:</label>
    <input type="text" id="usuario" placeholder="Digite seu nome"><br><br>

    <label for="matricula">Número de Matrícula:</label>
    <input type="text" id="matricula" placeholder="Digite sua matrícula"><br><br>

    <label for="data">Escolha a data:</label>
    <input type="date" id="data"><br><br>

    <button id="alternar-layout">Alternar Layout</button>

    <div class="raias" id="raias">
        <div class="raia" data-raia="1">Raia 1</div>
        <div class="raia" data-raia="2">Raia 2</div>
        <div class="raia" data-raia="3">Raia 3</div>
        <div class="raia" data-raia="4">Raia 4</div>
    </div>

    <div class="horarios">
        <button data-tempo="30">30 Minutos</button>
        <button data-tempo="60">1 Hora</button>
    </div>

    <button id="confirmar">Confirmar Reserva</button>

    <div id="resumo"></div>

    <h2>Reservas do Dia</h2>
    <div id="reservas-lista"></div>

    <script>
        let raiaSelecionada = null;
        let tempoSelecionado = null;
        let reservas = {};
        let layoutVertical = false;

        document.querySelectorAll('.raia').forEach(raia => {
            raia.addEventListener('click', () => {
                document.querySelectorAll('.raia').forEach(r => r.classList.remove('selected'));
                raia.classList.add('selected');
                raiaSelecionada = raia.getAttribute('data-raia');
            });
        });

        document.querySelectorAll('.horarios button').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('.horarios button').forEach(b => b.classList.remove('selected'));
                btn.classList.add('selected');
                tempoSelecionado = parseInt(btn.getAttribute('data-tempo'));
            });
        });

        document.getElementById('confirmar').addEventListener('click', () => {
            const usuario = document.getElementById('usuario').value.trim();
            const matricula = document.getElementById('matricula').value.trim();
            const data = document.getElementById('data').value;

            if (!usuario || !matricula || !data || !raiaSelecionada || !tempoSelecionado) {
                alert('Por favor, preencha todos os campos.');
                return;
            }

            if (!reservas[data]) {
                reservas[data] = { totalHoras: 0, usuarios: {} };
            }

            const totalHorasDia = reservas[data].totalHoras + tempoSelecionado / 60;
            const usuarioData = reservas[data].usuarios[matricula] || { nome: usuario, horasReservadas: 0, detalhes: [] };
            const horasUsuario = usuarioData.horasReservadas + tempoSelecionado / 60;

            if (totalHorasDia > 12) {
                alert('Não é possível exceder 12 horas de reservas no dia.');
                return;
            }

            if (horasUsuario > 1) {
                alert('Cada usuário pode reservar no máximo 1 hora por dia.');
                return;
            }

            reservas[data].totalHoras = totalHorasDia;
            usuarioData.horasReservadas = horasUsuario;
            usuarioData.detalhes.push(`Raia ${raiaSelecionada} - ${tempoSelecionado} min`);
            reservas[data].usuarios[matricula] = usuarioData;

            document.getElementById('resumo').innerText = `Reserva confirmada: ${usuario} (${matricula}) - Raia ${raiaSelecionada}, ${tempoSelecionado} min em ${data}`;

            atualizarListaReservas(data);
        });

        function atualizarListaReservas(data) {
            const lista = document.getElementById('reservas-lista');
            lista.innerHTML = '';

            if (reservas[data]) {
                for (const [matricula, usuarioData] of Object.entries(reservas[data].usuarios)) {
                    const item = document.createElement('div');
                    item.textContent = `${usuarioData.nome} (${matricula}): ${usuarioData.detalhes.join(', ')}`;
                    lista.appendChild(item);
                }
            }
        }

        document.getElementById('alternar-layout').addEventListener('click', () => {
            const raiasContainer = document.getElementById('raias');
            layoutVertical = !layoutVertical;

            if (layoutVertical) {
                raiasContainer.classList.add('vertical');
                for (let i = 5; i <= 8; i++) {
                    const novaRaia = document.createElement('div');
                    novaRaia.classList.add('raia');
                    novaRaia.setAttribute('data-raia', i);
                    novaRaia.innerText = `Raia ${i}`;
                    novaRaia.addEventListener('click', () => {
                        document.querySelectorAll('.raia').forEach(r => r.classList.remove('selected'));
                        novaRaia.classList.add('selected');
                        raiaSelecionada = novaRaia.getAttribute('data-raia');
                    });
                    raiasContainer.appendChild(novaRaia);
                }
            } else {
                raiasContainer.classList.remove('vertical');
                document.querySelectorAll('.raia').forEach(raia => {
                    if (parseInt(raia.getAttribute('data-raia')) > 4) {
                        raia.remove();
                    }
                });
            }
        });
    </script>
</body>
</html>

// =====================================================
// SCRIPT DO NÚCLEO DE TRIAGEM
// =====================================================

// FUNÇÕES BÁSICAS
function pegar(id) {
    return document.getElementById(id);
}

function valorCampo(id, padrao) {
    var campo = pegar(id);
    if (!campo) return padrao == null ? "" : padrao;
    return campo.value == null ? "" : String(campo.value).trim();
}

function pegarTodos(seletor) {
    return Array.from(document.querySelectorAll(seletor));
}

function criarId() {
    return Date.now().toString() +
        Math.random().toString(36).substring(2, 7);
}

function somenteNumeros(texto) {
    return (texto || "").replace(/\D/g, "");
}

function textoSeguro(texto) {
    if (texto == null) return "";

    return String(texto)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;");
}

function mostrarMensagem(mensagem) {
    var caixa = pegar("recado");

    if (!caixa) {
        console.log(mensagem);
        return;
    }

    caixa.textContent = mensagem;
    caixa.classList.add("aparece");

    setTimeout(function () {
        caixa.classList.remove("aparece");
    }, 3000);
}


// BANCO DE DADOS LOCAL
var banco = {
    pacientes: [],
    atendimentos: [],
    profissionais: [],
    sistema: []
};

function carregarBanco() {
    function lerLista(chave) {
        try {
            var valor = localStorage.getItem(chave);
            var dados = valor ? JSON.parse(valor) : [];
            return Array.isArray(dados) ? dados : [];
        } catch (erro) {
            console.warn("Erro ao ler " + chave, erro);
            return [];
        }
    }

    banco.pacientes = lerLista("triagem_pacientes");
    banco.atendimentos = lerLista("triagem_atendimentos");
    banco.profissionais = lerLista("triagem_profissionais");
    banco.sistema = lerLista("triagem_sistema");
}

function salvarBanco() {
    try {
        localStorage.setItem(
            "triagem_pacientes",
            JSON.stringify(banco.pacientes)
        );

        localStorage.setItem(
            "triagem_atendimentos",
            JSON.stringify(banco.atendimentos)
        );

        localStorage.setItem(
            "triagem_profissionais",
            JSON.stringify(banco.profissionais)
        );

        localStorage.setItem(
            "triagem_sistema",
            JSON.stringify(banco.sistema)
        );
    } catch (erro) {
        console.error("Erro ao salvar os dados", erro);
        mostrarMensagem("Não foi possível salvar os dados neste navegador.");
    }
}


// SENHAS — APENAS PARA PROTÓTIPO ACADÊMICO
function protegerSenha(senha) {
    var numero = 0;

    for (var i = 0; i < senha.length; i++) {
        numero =
            ((numero << 5) - numero) +
            senha.charCodeAt(i);

        numero = numero & numero;
    }

    return String(Math.abs(numero));
}


// CPF
function validarCPF(cpf) {
    cpf = somenteNumeros(cpf);
    return cpf.length === 11;
}

function formatarCPF(valor) {
    var cpf = somenteNumeros(valor).substring(0, 11);

    if (cpf.length > 9) {
        return cpf.replace(
            /(\d{3})(\d{3})(\d{3})(\d{2})/,
            "$1.$2.$3-$4"
        );
    }

    if (cpf.length > 6) {
        return cpf.replace(
            /(\d{3})(\d{3})(\d{1,3})/,
            "$1.$2.$3"
        );
    }

    if (cpf.length > 3) {
        return cpf.replace(
            /(\d{3})(\d{1,3})/,
            "$1.$2"
        );
    }

    return cpf;
}

function mascararCPF(cpf) {
    cpf = somenteNumeros(cpf);

    if (cpf.length !== 11) return "•••";

    return "•••." + cpf.substring(3, 6) + ".•••-••";
}

function mascararRG(rg) {
    if (!rg) return "—";
    return "••••" + rg.substring(rg.length - 2);
}


// DATA E HORÁRIO
function mostrarData(data) {
    if (!data) return "—";

    return new Date(data).toLocaleString("pt-BR", {
        day: "2-digit",
        month: "2-digit",
        year: "2-digit",
        hour: "2-digit",
        minute: "2-digit"
    });
}

function calcularIdade(nascimento) {
    if (!nascimento) return "—";

    var dataNascimento = new Date(nascimento);
    var hoje = new Date();

    var idade =
        hoje.getFullYear() - dataNascimento.getFullYear();

    var mes = hoje.getMonth() - dataNascimento.getMonth();

    if (
        mes < 0 ||
        (mes === 0 && hoje.getDate() < dataNascimento.getDate())
    ) {
        idade--;
    }

    return idade + " anos";
}

function calcularEspera(data) {
    var minutos = Math.floor((Date.now() - data) / 60000);

    if (minutos < 60) return minutos + " min";

    var horas = Math.floor(minutos / 60);
    var resto = minutos % 60;

    return horas + "h" + String(resto).padStart(2, "0");
}


// SESSÃO
var sessao = null;
var tipoProfissional = "enfermeiro";
var atendimentoSelecionado = null;
var dorEscolhida = null;
var riscoEscolhido = null;


// CLASSIFICAÇÃO DE RISCO — CINCO CORES
var riscos = [
    { chave: "azul", nome: "Azul · não urgente" },
    { chave: "verde", nome: "Verde · pouco urgente" },
    { chave: "amarelo", nome: "Amarelo · urgente" },
    { chave: "laranja", nome: "Laranja · muito urgente" },
    { chave: "vermelho", nome: "Vermelho · emergência" }
];

function nomeDoRisco(chave) {
    for (var i = 0; i < riscos.length; i++) {
        if (riscos[i].chave === chave) return riscos[i].nome;
    }

    return "Ainda não classificado";
}

function montarBotoesRisco(riscoAtual) {
    var html = "";

    riscos.forEach(function (risco) {
        html +=
            '<button type="button" data-risco="' +
            risco.chave +
            '" aria-pressed="' +
            (riscoAtual === risco.chave) +
            '">' +
            '<span class="bolinha ' +
            risco.chave +
            '"></span> ' +
            risco.nome +
            "</button>";
    });

    return html;
}

function prioridadeRisco(risco) {
    var prioridades = {
        vermelho: 1,
        laranja: 2,
        amarelo: 3,
        verde: 4,
        azul: 5
    };

    return prioridades[risco] || 6;
}


// TROCA DE TELAS
function mostrarTela(nome) {
    document.querySelectorAll(".tela").forEach(function (tela) {
        tela.classList.remove("ativa");
    });

    var telaSelecionada = pegar("tela-" + nome);

    if (telaSelecionada) {
        telaSelecionada.classList.add("ativa");
    }

    var topo = pegar("topo");

    if (topo) {
        topo.hidden = nome === "inicio";
    }

    window.scrollTo(0, 0);

    if (nome === "paciente-painel") {
        mostrarPainelPaciente();
    }

    if (nome === "prof-painel") {
        mostrarFila();
    }
}


// NAVEGAÇÃO
pegarTodos("[data-ir]").forEach(function (botao) {
    botao.addEventListener("click", function () {
        var destino = botao.dataset.ir;

        if (destino === "paciente") {
            mostrarTela("paciente-entrada");
        } else if (destino === "enfermeiro") {
            prepararLoginProfissional("enfermeiro");
        } else if (destino === "tecnico") {
            prepararLoginProfissional("tecnico");
        } else {
            mostrarTela(destino);
        }
    });
});

if (pegar("btnManual")) {
    pegar("btnManual").addEventListener("click", function () {
        mostrarTela("manual");
    });
}

if (pegar("voltarInicio")) {
    pegar("voltarInicio").addEventListener("click", function () {
        if (!sessao) {
            mostrarTela("inicio");
        } else if (sessao.papel === "paciente") {
            mostrarTela("paciente-painel");
        } else {
            mostrarTela("prof-painel");
        }
    });
}


// SAIR
function sairDoSistema() {
    sessao = null;
    atendimentoSelecionado = null;

    var quemSou = pegar("quemSou");
    if (quemSou) quemSou.textContent = "";

    mostrarTela("inicio");
    mostrarMensagem("Você saiu do sistema.");
}

if (pegar("btnSair")) {
    pegar("btnSair").addEventListener("click", sairDoSistema);
}


// TEMA
if (pegar("btnTema")) {
    pegar("btnTema").addEventListener("click", function () {
        var temaAtual =
            document.documentElement.getAttribute("data-theme");

        var novoTema = temaAtual === "dark" ? "light" : "dark";

        document.documentElement.setAttribute("data-theme", novoTema);
        localStorage.setItem("triagem_tema", novoTema);
    });
}


// ABAS
pegarTodos(".abas").forEach(function (grupo) {
    grupo.addEventListener("click", function (evento) {
        var botao = evento.target.closest("button[data-painel]");

        if (!botao) return;

        grupo.querySelectorAll("button").forEach(function (item) {
            item.setAttribute("aria-selected", item === botao);
        });

        var paineis =
            grupo.parentElement.querySelectorAll(":scope > .painel");

        paineis.forEach(function (painel) {
            painel.classList.remove("ativo");
        });

        var painelSelecionado = pegar(botao.dataset.painel);

        if (painelSelecionado) {
            painelSelecionado.classList.add("ativo");
        }
    });
});


// ESCALA DE DOR
var carasDor = [
    "😀", "🙂", "😊", "😐", "😕",
    "😣", "😖", "😫", "😭", "😡"
];

function descreverDor(numero) {
    if (numero <= 2) return "leve";
    if (numero <= 4) return "incômoda";
    if (numero <= 6) return "moderada";
    if (numero <= 8) return "forte";
    return "insuportável";
}

function montarEscalaDor() {
    var area = pegar("escalaDor");
    if (!area) return;

    area.innerHTML = "";

    for (var i = 0; i < 10; i++) {
        var botao = document.createElement("button");

        botao.type = "button";
        botao.className = "dor-item";
        botao.setAttribute("aria-pressed", "false");

        botao.innerHTML =
            "<span style='font-size:42px'>" +
            carasDor[i] +
            "</span><span>" + (i + 1) + "</span>";

        botao.addEventListener("click", function () {
            var botoes = document.querySelectorAll(
                "#escalaDor .dor-item"
            );

            botoes.forEach(function (item) {
                item.setAttribute("aria-pressed", "false");
            });

            this.setAttribute("aria-pressed", "true");

            dorEscolhida =
                Array.from(botoes).indexOf(this) + 1;

            var legenda = pegar("dorLegenda");

            if (legenda) {
                legenda.textContent =
                    "Dor " + dorEscolhida +
                    " de 10 — " + descreverDor(dorEscolhida);
            }
        });

        area.appendChild(botao);
    }
}


// FORMATAÇÃO AUTOMÁTICA DE CPF
[
    "c-cpf",
    "f-cpf",
    "f-login-cpf",
    "ac-cpf"
].forEach(function (idCampo) {
    var campo = pegar(idCampo);

    if (!campo) return;

    campo.addEventListener("input", function () {
        campo.value = formatarCPF(campo.value);
    });
});
// =====================================================
// CADASTRO DO PACIENTE
// =====================================================

if (pegar("formCadastro")) {
    pegar("formCadastro").addEventListener("submit", function (evento) {
        evento.preventDefault();

        var formulario = pegar("formCadastro");
        var erro = pegar("c-erro");

        if (!erro) {
            erro = document.createElement("p");
            erro.id = "c-erro";
            erro.className = "erro";
            formulario.prepend(erro);
        }

        erro.textContent = "";

        var camposObrigatorios = [
            ["c-nome", "nome completo"],
            ["c-nasc", "data de nascimento"],
            ["c-cpf", "CPF"],
            ["c-rg", "RG"],
            ["c-tel", "telefone"],
            ["c-alergia", "alergias"],
            ["c-em-nome", "nome do contato de emergência"],
            ["c-em-par", "parentesco do contato"],
            ["c-em-tel", "telefone do contato"]
        ];

        for (var i = 0; i < camposObrigatorios.length; i++) {
            var campo = pegar(camposObrigatorios[i][0]);

            if (!campo || !campo.value.trim()) {
                erro.textContent =
                    "Preencha o campo: " +
                    camposObrigatorios[i][1] + ".";

                if (campo) campo.focus();
                return;
            }
        }

        var cpf = somenteNumeros(valorCampo("c-cpf"));

        if (!validarCPF(cpf)) {
            erro.textContent = "O CPF precisa ter 11 números.";
            pegar("c-cpf").focus();
            return;
        }

        var senha = valorCampo("c-senha");
        var senha2 = valorCampo("c-senha2");

        if (senha.length < 6) {
            erro.textContent =
                "A senha precisa ter pelo menos 6 caracteres.";
            return;
        }

        if (senha !== senha2) {
            erro.textContent = "As senhas não conferem.";
            return;
        }

        var consentimento = pegar("c-consent");

        if (!consentimento || !consentimento.checked) {
            erro.textContent =
                "É necessário aceitar o consentimento para continuar.";
            return;
        }

        var doencas = [];

        pegarTodos("#c-doencas input:checked").forEach(function (checkbox) {
            doencas.push(checkbox.value);
        });

        var paciente = {
            id: criarId(),
            nome: valorCampo("c-nome"),
            nomeSocial: valorCampo("c-social"),
            nascimento: valorCampo("c-nasc"),
            cpf: cpf,
            rg: valorCampo("c-rg"),
            sus: valorCampo("c-sus"),
            govbr: valorCampo("c-govbr"),
            telefone: valorCampo("c-tel"),
            sangue: valorCampo("c-sangue"),

            endereco: {
                cep: valorCampo("c-cep"),
                rua: valorCampo("c-rua"),
                numero: valorCampo("c-num"),
                bairro: valorCampo("c-bairro"),
                cidade: valorCampo("c-cidade"),
                estado: valorCampo("c-uf").toUpperCase()
            },

            alergias: valorCampo("c-alergia"),
            doencas: doencas,
            outras: valorCampo("c-outras"),
            medicamentos: valorCampo("c-medic"),

            emergencia: {
                nome: valorCampo("c-em-nome"),
                parentesco: valorCampo("c-em-par"),
                telefone: valorCampo("c-em-tel")
            },

            senha: protegerSenha(senha),

            consentimento: {
                aceito: true,
                quando: Date.now(),
                versao: "1.0"
            },

            criadoEm: Date.now()
        };

        // Atualiza o cadastro existente pelo CPF,
        // preservando o histórico de atendimentos.
        var antigo = banco.pacientes.find(function (item) {
            return item.cpf === cpf;
        });

        if (antigo) {
            paciente.id = antigo.id;
            paciente.criadoEm = antigo.criadoEm;

            banco.pacientes[
                banco.pacientes.indexOf(antigo)
            ] = paciente;
        } else {
            banco.pacientes.push(paciente);
        }

        salvarBanco();

        sessao = {
            papel: "paciente",
            id: paciente.id,
            nome: paciente.nomeSocial || paciente.nome
        };

        registrarAcao(
            antigo ? "Atualizou o cadastro" : "Criou o cadastro",
            paciente.id
        );

        formulario.reset();

        mostrarMensagem(
            "Cadastro salvo! Agora você pode informar seus sintomas."
        );

        entrarPaciente();
    });
}


// =====================================================
// PAINEL DO PACIENTE
// =====================================================

function encontrarPaciente(id) {
    return banco.pacientes.find(function (paciente) {
        return paciente.id === id;
    });
}

function entrarPaciente() {
    if (!sessao) return;

    var paciente = encontrarPaciente(sessao.id);
    if (!paciente) return;

    var quemSou = pegar("quemSou");
    if (quemSou) {
        quemSou.textContent =
            "Paciente · " + (paciente.nomeSocial || paciente.nome);
    }

    var saudacao = pegar("pac-saudacao");
    if (saudacao) {
        saudacao.textContent =
            "Olá, " + (paciente.nomeSocial || paciente.nome).split(" ")[0];
    }

    abrirAbaInicialPaciente();
    mostrarTela("paciente-painel");
}

function abrirAbaInicialPaciente() {
    pegarTodos('button[data-painel^="pp-"]').forEach(function (botao) {
        botao.setAttribute(
            "aria-selected",
            botao.dataset.painel === "pp-atend"
        );
    });

    ["pp-atend", "pp-dados", "pp-hist", "pp-priv"].forEach(
        function (idPainel) {
            var painel = pegar(idPainel);

            if (painel) {
                painel.classList.toggle(
                    "ativo",
                    idPainel === "pp-atend"
                );
            }
        }
    );
}


// =====================================================
// ENVIAR ATENDIMENTO
// =====================================================

if (pegar("formAtend")) {
    pegar("formAtend").addEventListener("submit", function (evento) {
        evento.preventDefault();

        if (!sessao || sessao.papel !== "paciente") {
            mostrarMensagem("Faça seu cadastro antes de enviar a ficha.");
            return;
        }

        var erro = pegar("a-erro");
        if (erro) erro.textContent = "";

        var motivo = valorCampo("a-motivo");
        var sintomas = valorCampo("a-sintomas");

        if (!motivo || !sintomas) {
            if (erro) erro.textContent =
                "Informe o motivo da consulta e os sintomas.";
            return;
        }

        if (!dorEscolhida) {
            if (erro) erro.textContent =
                "Selecione a intensidade da dor.";
            return;
        }

        var numero = banco.atendimentos.length + 1;
        var senhaAtendimento =
            "T" + String(numero).padStart(3, "0");

        var atendimento = {
            id: criarId(),
            pacienteId: sessao.id,
            senhaChamada: senhaAtendimento,
            abertoEm: Date.now(),
            motivo: motivo,
            inicio: valorCampo("a-inicio"),
            sintomas: sintomas,
            dor: dorEscolhida,
            risco: null,
            riscoConfirmado: false,
            status: "aguardando",

            acompanhante: {
                nome: valorCampo("ac-nome"),
                parentesco: valorCampo("ac-par"),
                rg: valorCampo("ac-rg"),
                cpf: somenteNumeros(valorCampo("ac-cpf")),
                telefone: valorCampo("ac-tel"),
                endereco: valorCampo("ac-end")
            },

            sinais: {},
            anotacoes: "",

            eventos: [{
                quando: Date.now(),
                texto: "Ficha aberta pelo paciente"
            }]
        };

        banco.atendimentos.push(atendimento);
        salvarBanco();

        registrarAcao("Abriu atendimento", senhaAtendimento);

        pegar("formAtend").reset();
        dorEscolhida = null;

        pegarTodos("#escalaDor .dor-item").forEach(function (botao) {
            botao.setAttribute("aria-pressed", "false");
        });

        var legenda = pegar("dorLegenda");
        if (legenda) {
            legenda.textContent = "Nenhuma carinha selecionada";
        }

        mostrarMensagem("Ficha enviada! Sua senha é " + senhaAtendimento);
        mostrarPainelPaciente();
    });
}


// =====================================================
// ATUALIZA PAINEL DO PACIENTE
// =====================================================

function mostrarPainelPaciente() {
    if (!sessao || sessao.papel !== "paciente") return;

    var paciente = encontrarPaciente(sessao.id);
    if (!paciente) return;

    var meusAtendimentos = banco.atendimentos
        .filter(function (item) {
            return item.pacienteId === paciente.id;
        })
        .sort(function (a, b) {
            return b.abertoEm - a.abertoEm;
        });

    var aberto = meusAtendimentos.find(function (item) {
        return item.status !== "finalizado";
    });

    mostrarAtendimentoAberto(aberto);
    mostrarDadosPaciente(paciente);
    mostrarHistoricoPaciente(meusAtendimentos);
    mostrarPrivacidade(paciente, meusAtendimentos);
}

function mostrarAtendimentoAberto(atendimento) {
    var area = pegar("atendAberto");
    var formulario = pegar("formAtend");

    if (!area || !formulario) return;

    if (!atendimento) {
        area.innerHTML = "";
        formulario.style.display = "block";
        return;
    }

    formulario.style.display = "none";

    var riscoTexto = atendimento.risco
        ? "Classificação: " + nomeDoRisco(atendimento.risco)
        : "Aguardando classificação da enfermagem.";

    area.innerHTML = `
        <div class="cartao" style="text-align:center">
            <h2>Cadastro concluído</h2>
            <p>Suas informações foram enviadas para a equipe.</p>
            <p>Sua senha de atendimento é:
                <b>${textoSeguro(atendimento.senhaChamada)}</b>
            </p>
            <p>Tempo de espera:
                ${calcularEspera(atendimento.abertoEm)}
            </p>
            <p>${textoSeguro(riscoTexto)}</p>
            <p>
                Aguarde ser chamado(a). Se você piorar enquanto espera,
                avise a equipe de saúde.
            </p>
            <button class="btn" id="btnConcluirVisita" type="button">
                Voltar ao início
            </button>
        </div>
    `;

    var voltar = pegar("btnConcluirVisita");
    if (voltar) {
        voltar.addEventListener("click", sairDoSistema);
    }
}


// =====================================================
// DADOS DO PACIENTE
// =====================================================

function linhaTabela(nome, valor) {
    return `
        <tr>
            <th>${textoSeguro(nome)}</th>
            <td>${textoSeguro(valor)}</td>
        </tr>
    `;
}

function mostrarDadosPaciente(paciente) {
    var tabela = pegar("tabelaMeusDados");
    if (!tabela) return;

    var endereco = paciente.endereco || {};

    var enderecoTexto = [
        endereco.rua,
        endereco.numero,
        endereco.bairro,
        endereco.cidade,
        endereco.estado
    ].filter(Boolean).join(", ");

    tabela.innerHTML = `
        <tbody>
            ${linhaTabela("Nome", paciente.nome)}
            ${linhaTabela("Nome social", paciente.nomeSocial || "—")}
            ${linhaTabela(
                "Nascimento",
                (paciente.nascimento || "—") +
                " (" + calcularIdade(paciente.nascimento) + ")"
            )}
            ${linhaTabela("CPF", formatarCPF(paciente.cpf))}
            ${linhaTabela("RG", paciente.rg)}
            ${linhaTabela("Cartão SUS", paciente.sus || "—")}
            ${linhaTabela("Telefone", paciente.telefone)}
            ${linhaTabela("Tipo sanguíneo", paciente.sangue || "Não informado")}
            ${linhaTabela("Endereço", enderecoTexto || "—")}
            ${linhaTabela("Alergias", paciente.alergias)}
            ${linhaTabela(
                "Doenças",
                paciente.doencas && paciente.doencas.length
                    ? paciente.doencas.join(", ")
                    : "Nenhuma marcada"
            )}
            ${linhaTabela("Outras doenças", paciente.outras || "—")}
            ${linhaTabela("Medicamentos", paciente.medicamentos || "—")}
            ${linhaTabela(
                "Contato de emergência",
                paciente.emergencia
                    ? paciente.emergencia.nome + " (" +
                      paciente.emergencia.parentesco + ") - " +
                      paciente.emergencia.telefone
                    : "—"
            )}
        </tbody>
    `;
}


// =====================================================
// HISTÓRICO DO PACIENTE
// =====================================================

function mostrarHistoricoPaciente(atendimentos) {
    var area = pegar("histPaciente");
    if (!area) return;

    if (!atendimentos.length) {
        area.innerHTML =
            '<div class="vazio">Você ainda não possui atendimentos.</div>';
        return;
    }

    var html = `
        <div class="rolagem">
            <table>
                <thead>
                    <tr>
                        <th>Data</th>
                        <th>Senha</th>
                        <th>Motivo</th>
                        <th>Dor</th>
                        <th>Risco</th>
                        <th>Situação</th>
                    </tr>
                </thead>
                <tbody>
    `;

    atendimentos.forEach(function (atendimento) {
        html += `
            <tr>
                <td>${mostrarData(atendimento.abertoEm)}</td>
                <td>${textoSeguro(atendimento.senhaChamada)}</td>
                <td>${textoSeguro(atendimento.motivo)}</td>
                <td>${textoSeguro(atendimento.dor)}/10</td>
                <td>${textoSeguro(nomeDoRisco(atendimento.risco))}</td>
                <td>${textoSeguro(atendimento.status)}</td>
            </tr>
        `;
    });

    html += "</tbody></table></div>";
    area.innerHTML = html;
}


// =====================================================
// PRIVACIDADE E AUDITORIA DO PACIENTE
// =====================================================

function mostrarPrivacidade(paciente, atendimentos) {
    var consentimento = paciente.consentimento;
    var texto = pegar("consentTexto");

    if (texto && consentimento && consentimento.aceito) {
        texto.textContent =
            "Você aceitou o uso dos seus dados em " +
            mostrarData(consentimento.quando) +
            ". Versão do termo: " + consentimento.versao + ".";
    }

    mostrarAuditoriaDoPaciente(paciente, atendimentos);
}

function mostrarAuditoriaDoPaciente(paciente, atendimentos) {
    var tabela = pegar("tabelaAuditoria");
    if (!tabela) return;

    var corpo = tabela.querySelector("tbody");
    if (!corpo) return;

    var alvos = [paciente.id];

    atendimentos.forEach(function (atendimento) {
        alvos.push(atendimento.senhaChamada);
    });

    var eventos = banco.sistema
        .filter(function (evento) {
            return alvos.indexOf(evento.alvo) >= 0;
        })
        .slice(-30)
        .reverse();

    if (!eventos.length) {
        corpo.innerHTML =
            '<tr><td colspan="4" class="vazio">' +
            "Nenhum acesso registrado ainda." +
            "</td></tr>";
        return;
    }

    corpo.innerHTML = eventos.map(function (evento) {
        return `
            <tr>
                <td>${mostrarData(evento.quando)}</td>
                <td>${textoSeguro(evento.quem)}</td>
                <td>${textoSeguro(evento.papel)}</td>
                <td>${textoSeguro(evento.acao)}</td>
            </tr>
        `;
    }).join("");
}


// =====================================================
// EXPORTAR OS DADOS DO PACIENTE
// =====================================================

if (pegar("btnExportar")) {
    pegar("btnExportar").addEventListener("click", function () {
        if (!sessao) return;

        var paciente = encontrarPaciente(sessao.id);
        if (!paciente) return;

        var atendimentos = banco.atendimentos.filter(function (item) {
            return item.pacienteId === paciente.id;
        });

        var dados = {
            paciente: paciente,
            atendimentos: atendimentos,
            geradoEm: new Date().toISOString()
        };

        var blob = new Blob(
            [JSON.stringify(dados, null, 2)],
            { type: "application/json" }
        );

        var url = URL.createObjectURL(blob);
        var link = document.createElement("a");

        link.href = url;
        link.download = "meus-dados-triagem.json";
        link.click();

        URL.revokeObjectURL(url);
        registrarAcao("Exportou os próprios dados", "");
        mostrarMensagem("Seus dados foram exportados.");
    });
}


// =====================================================
// EXCLUIR CADASTRO DO PACIENTE
// =====================================================

if (pegar("btnExcluir")) {
    pegar("btnExcluir").addEventListener("click", function () {
        if (!sessao) return;

        var confirmar = confirm(
            "Isso vai excluir seu cadastro e seus atendimentos. Deseja continuar?"
        );

        if (!confirmar) return;

        banco.atendimentos = banco.atendimentos.filter(function (item) {
            return item.pacienteId !== sessao.id;
        });

        banco.pacientes = banco.pacientes.filter(function (item) {
            return item.id !== sessao.id;
        });

        salvarBanco();
        sessao = null;

        mostrarMensagem("Seus dados foram excluídos.");
        mostrarTela("inicio");
    });
}
// =====================================================
// REGISTRO DE AÇÕES (AUDITORIA)
// =====================================================

function registrarAcao(acao, alvo) {
    banco.sistema.push({
        quando: Date.now(),
        quem: sessao ? sessao.nome : "Visitante",
        papel: sessao ? sessao.papel : "visitante",
        acao: acao,
        alvo: alvo || ""
    });

    if (banco.sistema.length > 200) {
        banco.sistema = banco.sistema.slice(-200);
    }

    salvarBanco();
}


// =====================================================
// PREPARAR LOGIN PROFISSIONAL
// =====================================================

function prepararLoginProfissional(tipo) {
    tipoProfissional = tipo;

    var titulo = pegar("prof-titulo");
    var subtitulo = pegar("prof-sub");

    if (titulo) {
        titulo.textContent = tipo === "enfermeiro"
            ? "Área do enfermeiro"
            : "Área do técnico em enfermagem";
    }

    if (subtitulo) {
        subtitulo.textContent = tipo === "enfermeiro"
            ? "Entre com seu cadastro profissional para acessar a fila e classificar o risco."
            : "Entre com seu cadastro profissional para registrar sinais vitais e acompanhar a fila.";
    }

    ["f-erro", "f-login-erro"].forEach(function (id) {
        var campo = pegar(id);
        if (campo) campo.textContent = "";
    });

    [
        "f-login-nome",
        "f-login-cpf",
        "f-login-coren",
        "f-login-senha"
    ].forEach(function (id) {
        var campo = pegar(id);
        if (campo) campo.value = "";
    });

    mostrarTela("prof-entrada");
}


// =====================================================
// CADASTRO DO PROFISSIONAL
// =====================================================

if (pegar("btnCadastrarProf")) {
    pegar("btnCadastrarProf").addEventListener("click", function () {
        var erro = pegar("f-erro");

        if (!erro) {
            mostrarMensagem("Não foi encontrado o campo de mensagens do cadastro profissional.");
            return;
        }

        erro.textContent = "";

        var nome = valorCampo("f-nome");
        var cpf = somenteNumeros(valorCampo("f-cpf"));
        var rg = valorCampo("f-rg");
        var coren = valorCampo("f-coren");
        var senha = valorCampo("f-senha");
        var confirmaSenha = valorCampo("f-confirma-senha");

        if (!nome || !cpf || !rg || !coren || !senha || !confirmaSenha) {
            erro.textContent = "Preencha todos os campos do cadastro.";
            return;
        }

        if (!validarCPF(cpf)) {
            erro.textContent = "O CPF precisa ter 11 números.";
            return;
        }

        if (senha.length < 6) {
            erro.textContent = "A senha precisa ter pelo menos 6 caracteres.";
            return;
        }

        if (senha !== confirmaSenha) {
            erro.textContent = "As senhas não conferem.";
            return;
        }

        var existente = banco.profissionais.find(function (item) {
            return (
                somenteNumeros(item.cpf) === cpf ||
                String(item.coren || "").trim().toLowerCase() === coren.toLowerCase()
            );
        });

        if (existente) {
            erro.textContent =
                "Já existe um cadastro com esse CPF ou número de COREN.";
            return;
        }

        var profissional = {
            id: criarId(),
            nome: nome,
            cpf: cpf,
            rg: rg,
            coren: coren,
            papel: tipoProfissional,
            senha: protegerSenha(senha),
            criadoEm: Date.now()
        };

        banco.profissionais.push(profissional);
        salvarBanco();

        // Preenche os campos de login, caso existam.
        var loginNome = pegar("f-login-nome");
        var loginCPF = pegar("f-login-cpf");
        var loginCoren = pegar("f-login-coren");

        if (loginNome) loginNome.value = nome;
        if (loginCPF) loginCPF.value = formatarCPF(cpf);
        if (loginCoren) loginCoren.value = coren;

        var loginSenha = pegar("f-login-senha");
        if (loginSenha) loginSenha.value = "";

        erro.textContent =
            "Cadastro criado! Entre com seus dados e sua senha.";

        [
            "f-nome",
            "f-cpf",
            "f-rg",
            "f-coren",
            "f-senha",
            "f-confirma-senha"
        ].forEach(function (id) {
            var campo = pegar(id);
            if (campo) campo.value = "";
        });

        // Se a tela tiver abas profissionais, abre a aba de login.
        var botaoEntrar = pegarTodos(
            'button[data-painel="prof-entrar"]'
        )[0];

        var painelEntrar = pegar("prof-entrar");
        var painelCriar = pegar("prof-criar");

        if (botaoEntrar) {
            botaoEntrar.click();
        } else {
            if (painelCriar) painelCriar.classList.remove("ativo");
            if (painelEntrar) painelEntrar.classList.add("ativo");
        }

        mostrarMensagem("Cadastro profissional criado com sucesso.");
    });
}


// =====================================================
// LOGIN PROFISSIONAL
// =====================================================

if (pegar("btnEntrarProf")) {
    pegar("btnEntrarProf").addEventListener("click", function () {
        var erro = pegar("f-login-erro");

        if (!erro) {
            mostrarMensagem("Não foi encontrado o campo de erro do login.");
            return;
        }

        erro.textContent = "";

        var senha = valorCampo("f-login-senha");
        var campoCPF = pegar("f-login-cpf");
        var profissional = null;

        if (campoCPF) {
            var cpf = somenteNumeros(campoCPF.value);

            if (!cpf || !senha) {
                erro.textContent = "Informe o CPF e a senha.";
                return;
            }

            profissional = banco.profissionais.find(function (item) {
                return somenteNumeros(item.cpf) === cpf;
            });
        } else {
            var nome = valorCampo("f-login-nome");
            var coren = valorCampo("f-login-coren");

            if (!nome || !coren || !senha) {
                erro.textContent =
                    "Informe o nome completo, o COREN e a senha.";
                return;
            }

            profissional = banco.profissionais.find(function (item) {
                return (
                    String(item.nome || "").trim().toLowerCase() === nome.toLowerCase() &&
                    String(item.coren || "").trim().toLowerCase() === coren.toLowerCase()
                );
            });
        }

        if (!profissional) {
            erro.textContent =
                "Cadastro não encontrado. Faça o cadastro profissional primeiro.";
            return;
        }

        if (profissional.papel !== tipoProfissional) {
            erro.textContent =
                "Este cadastro pertence à outra área profissional.";
            return;
        }

        if (profissional.senha !== protegerSenha(senha)) {
            erro.textContent = "Senha incorreta.";
            return;
        }

        sessao = {
            papel: profissional.papel,
            id: profissional.id,
            nome: profissional.nome,
            coren: profissional.coren
        };

        var quemSou = pegar("quemSou");

        if (quemSou) {
            quemSou.textContent =
                (profissional.papel === "enfermeiro" ? "Enf. " : "Téc. ") +
                profissional.nome +
                " · COREN " +
                profissional.coren;
        }

        registrarAcao("Entrou no sistema", "");
        mostrarTela("prof-painel");
    });
}


// =====================================================
// FILA DE TRIAGEM
// =====================================================

function mostrarFila() {
    if (!sessao || sessao.papel === "paciente") return;

    // Atualiza os dados salvos por outras abas do mesmo navegador.
    carregarBanco();

    var aviso = pegar("avisoPerfil");
    var ehEnfermeiro = sessao.papel === "enfermeiro";

    if (aviso) {
        aviso.textContent = ehEnfermeiro
            ? "Você pode consultar a ficha, registrar sinais vitais, confirmar a classificação de risco e finalizar o atendimento."
            : "Perfil técnico: registre os sinais vitais disponíveis e sugira uma classificação para confirmação do enfermeiro.";
    }

    var controleFinalizados = pegar("verFinalizados");
    var mostrarFinalizados =
        controleFinalizados ? controleFinalizados.checked : false;

    var lista = banco.atendimentos.filter(function (atendimento) {
        return mostrarFinalizados ||
            atendimento.status !== "finalizado";
    });

    lista.sort(function (a, b) {
        var prioridadeA = prioridadeRisco(a.risco);
        var prioridadeB = prioridadeRisco(b.risco);

        if (prioridadeA !== prioridadeB) {
            return prioridadeA - prioridadeB;
        }

        return a.abertoEm - b.abertoEm;
    });

    var aguardando = banco.atendimentos.filter(function (item) {
        return item.status !== "finalizado";
    }).length;

    var semClassificacao = banco.atendimentos.filter(function (item) {
        return item.status !== "finalizado" && !item.risco;
    }).length;

    var resumo = pegar("resumoFila");

    if (resumo) {
        resumo.textContent = aguardando === 0
            ? "Ninguém está aguardando."
            : aguardando + " pessoas aguardando · " +
              semClassificacao + " sem classificação";
    }

    var tabela = pegar("tabelaFila");
    if (!tabela) return;

    var corpo = tabela.querySelector("tbody");
    if (!corpo) return;

    if (!lista.length) {
        corpo.innerHTML = `
            <tr>
                <td colspan="4" class="vazio">
                    Nenhuma ficha aberta.
                </td>
            </tr>
        `;
        return;
    }

    corpo.innerHTML = "";

    lista.forEach(function (atendimento) {
        var paciente = encontrarPaciente(atendimento.pacienteId);
        if (!paciente) return;

        var linha = document.createElement("tr");
        linha.className = "clicavel";

        if (atendimentoSelecionado === atendimento.id) {
            linha.classList.add("selecionada");
        }

        var risco = atendimento.risco || "nenhum";

        var confirmacao = atendimento.risco &&
            !atendimento.riscoConfirmado
            ? "<br><span class='etiqueta'>a confirmar</span>"
            : "";

        linha.innerHTML = `
            <td>
                <span class="bolinha ${textoSeguro(risco)}"
                      title="${textoSeguro(nomeDoRisco(atendimento.risco))}">
                </span>
                ${confirmacao}
            </td>
            <td>
                <b>${textoSeguro(paciente.nomeSocial || paciente.nome)}</b>
                <br>
                <span>
                    ${textoSeguro(atendimento.senhaChamada)}
                    · ${textoSeguro(calcularIdade(paciente.nascimento))}
                </span>
            </td>
            <td>${textoSeguro(atendimento.dor)}/10</td>
            <td>
                ${atendimento.status === "finalizado"
                    ? "Finalizado"
                    : calcularEspera(atendimento.abertoEm)}
            </td>
        `;

        linha.addEventListener("click", function () {
            abrirFicha(atendimento.id);
        });

        corpo.appendChild(linha);
    });
}

if (pegar("verFinalizados")) {
    pegar("verFinalizados").addEventListener("change", mostrarFila);
}


// =====================================================
// ABRIR FICHA DO PACIENTE
// =====================================================

function abrirFicha(idAtendimento) {
    var atendimento = banco.atendimentos.find(function (item) {
        return item.id === idAtendimento;
    });

    if (!atendimento) return;

    var paciente = encontrarPaciente(atendimento.pacienteId);
    if (!paciente) return;

    atendimentoSelecionado = idAtendimento;

    var ehEnfermeiro = sessao && sessao.papel === "enfermeiro";

    registrarAcao("Abriu a ficha do paciente", atendimento.senhaChamada);

    var sinais = atendimento.sinais || {};
    var acompanhante = atendimento.acompanhante || {};
    var endereco = paciente.endereco || {};

    var cpfMostrar = ehEnfermeiro
        ? formatarCPF(paciente.cpf)
        : mascararCPF(paciente.cpf);

    var rgMostrar = ehEnfermeiro
        ? paciente.rg
        : mascararRG(paciente.rg);

    var sistolica = sinais.sistolica || "";
    var diastolica = sinais.diastolica || "";

    // Compatibilidade com fichas antigas.
    if (!sistolica && sinais.pa && sinais.pa.indexOf("/") > -1) {
        sistolica = sinais.pa.split("/")[0];
        diastolica = sinais.pa.split("/")[1];
    }

    var campoFrequencia = ehEnfermeiro
        ? `
            <div class="campo">
                <label for="s-fc">Frequência cardíaca (bpm)</label>
                <input id="s-fc" type="number" min="0" max="300"
                       value="${textoSeguro(sinais.fc || "")}">
            </div>
        `
        : "";

    var avisoFrequencia = ehEnfermeiro
        ? ""
        : "<p>A frequência cardíaca é registrada na área do enfermeiro.</p>";

    var textoRisco = atendimento.risco
        ? "Risco atual: " + nomeDoRisco(atendimento.risco) +
          (atendimento.riscoConfirmado
              ? " · confirmado pelo enfermeiro"
              : " · aguardando confirmação do enfermeiro")
        : "Ainda sem classificação.";

    var ficha = pegar("fichaPaciente");
    if (!ficha) return;

    ficha.innerHTML = `
        <div class="cartao">
            <h2>${textoSeguro(paciente.nomeSocial || paciente.nome)}</h2>
            <p>Senha: <b>${textoSeguro(atendimento.senhaChamada)}</b></p>
            <p>Dor: <b>${textoSeguro(atendimento.dor)}/10</b></p>

            ${paciente.alergias && !/nenhuma/i.test(paciente.alergias)
                ? `<div class="aviso alerta">
                    <b>Alergia:</b> ${textoSeguro(paciente.alergias)}
                   </div>`
                : ""}

            <div class="rolagem">
                <table class="ficha">
                    <tbody>
                        ${linhaTabela("Motivo da vinda", atendimento.motivo)}
                        ${linhaTabela("Quando começou", atendimento.inicio || "—")}
                        ${linhaTabela("Sintomas", atendimento.sintomas)}
                        ${linhaTabela("CPF", cpfMostrar)}
                        ${linhaTabela("RG", rgMostrar)}
                        ${ehEnfermeiro
                            ? linhaTabela("Cartão SUS", paciente.sus || "—")
                            : ""}
                        ${linhaTabela("Nascimento", paciente.nascimento)}
                        ${linhaTabela("Tipo sanguíneo", paciente.sangue || "Não informado")}
                        ${linhaTabela(
                            "Doenças",
                            paciente.doencas && paciente.doencas.length
                                ? paciente.doencas.join(", ")
                                : "Nenhuma"
                        )}
                        ${linhaTabela("Medicamentos", paciente.medicamentos || "—")}
                        ${linhaTabela("Telefone", paciente.telefone || "—")}
                        ${ehEnfermeiro
                            ? linhaTabela(
                                "Endereço",
                                [
                                    endereco.rua,
                                    endereco.numero,
                                    endereco.bairro,
                                    endereco.cidade,
                                    endereco.estado
                                ].filter(Boolean).join(", ") || "—"
                            )
                            : ""}
                    </tbody>
                </table>
            </div>

            <h3>Acompanhante</h3>
            <div class="rolagem">
                <table class="ficha">
                    <tbody>
                        ${linhaTabela("Nome", acompanhante.nome || "Nenhum")}
                        ${linhaTabela("Parentesco", acompanhante.parentesco || "—")}
                        ${linhaTabela(
                            "RG",
                            ehEnfermeiro
                                ? acompanhante.rg || "—"
                                : mascararRG(acompanhante.rg)
                        )}
                        ${linhaTabela(
                            "CPF",
                            ehEnfermeiro
                                ? formatarCPF(acompanhante.cpf)
                                : mascararCPF(acompanhante.cpf)
                        )}
                        ${linhaTabela("Telefone", acompanhante.telefone || "—")}
                    </tbody>
                </table>
            </div>
        </div>

        <div class="cartao">
            <h3>Sinais vitais e medidas</h3>

            <div class="grade">
                <div class="campo">
                    <label for="s-sis">Pressão sistólica (máxima)</label>
                    <input id="s-sis" type="number" min="0" max="300"
                           value="${textoSeguro(sistolica)}">
                </div>

                <div class="campo">
                    <label for="s-dia">Pressão diastólica (mínima)</label>
                    <input id="s-dia" type="number" min="0" max="200"
                           value="${textoSeguro(diastolica)}">
                </div>

                ${campoFrequencia}

                <div class="campo">
                    <label for="s-fr">Frequência respiratória</label>
                    <input id="s-fr" value="${textoSeguro(sinais.fr || "")}">
                </div>

                <div class="campo">
                    <label for="s-temp">Temperatura</label>
                    <input id="s-temp" value="${textoSeguro(sinais.temp || "")}">
                </div>

                <div class="campo">
                    <label for="s-spo2">Saturação (%)</label>
                    <input id="s-spo2" value="${textoSeguro(sinais.spo2 || "")}">
                </div>

                <div class="campo">
                    <label for="s-glic">Glicemia</label>
                    <input id="s-glic" value="${textoSeguro(sinais.glicemia || "")}">
                </div>

                <div class="campo">
                    <label for="s-peso">Peso</label>
                    <input id="s-peso" value="${textoSeguro(sinais.peso || "")}">
                </div>
            </div>

            ${avisoFrequencia}

            <h3>Classificação de risco</h3>
            <div class="selbolinha" id="selRisco">
                ${montarBotoesRisco(atendimento.risco)}
            </div>

            <p>${textoSeguro(textoRisco)}</p>

            <div class="campo">
                <label for="s-notas">Anotações da enfermagem</label>
                <textarea id="s-notas">${textoSeguro(atendimento.anotacoes || "")}</textarea>
            </div>

            <div class="linha-botoes">
                <button class="btn" id="btnSalvarFicha" type="button">
                    Salvar ficha
                </button>

                ${ehEnfermeiro && atendimento.status !== "finalizado"
                    ? `<button class="btn neutro" id="btnFinalizar" type="button">
                        Finalizar atendimento
                       </button>`
                    : ""}
            </div>
        </div>
    `;

    configurarRisco(atendimento);

    var salvar = pegar("btnSalvarFicha");
    if (salvar) {
        salvar.addEventListener("click", function () {
            salvarFicha(atendimento.id);
        });
    }

    var finalizar = pegar("btnFinalizar");
    if (finalizar) {
        finalizar.addEventListener("click", function () {
            finalizarAtendimento(atendimento.id);
        });
    }
}
// =====================================================
// CONFIGURAR CLASSIFICAÇÃO DE RISCO
// =====================================================

function configurarRisco(atendimento) {
    riscoEscolhido = atendimento.risco || null;

    pegarTodos("#selRisco button").forEach(function (botao) {
        botao.addEventListener("click", function () {
            riscoEscolhido = botao.dataset.risco;

            pegarTodos("#selRisco button").forEach(function (outro) {
                outro.setAttribute(
                    "aria-pressed",
                    outro.dataset.risco === riscoEscolhido
                );
            });
        });
    });
}


// =====================================================
// SALVAR FICHA DO ATENDIMENTO
// =====================================================

function salvarFicha(idAtendimento) {
    var atendimento = banco.atendimentos.find(function (item) {
        return item.id === idAtendimento;
    });

    if (!atendimento) return;

    var sistolica = valorCampo("s-sis");
    var diastolica = valorCampo("s-dia");

    if (
        sistolica &&
        diastolica &&
        Number(diastolica) >= Number(sistolica)
    ) {
        mostrarMensagem(
            "A pressão diastólica precisa ser menor que a sistólica."
        );
        return;
    }

    var sinaisAntigos = atendimento.sinais || {};
    var campoFC = pegar("s-fc");

    atendimento.sinais = {
        sistolica: sistolica,
        diastolica: diastolica,

        pa: sistolica && diastolica
            ? sistolica + "/" + diastolica
            : "",

        fc: campoFC
            ? campoFC.value.trim()
            : (sinaisAntigos.fc || ""),

        fr: valorCampo("s-fr"),
        temp: valorCampo("s-temp"),
        spo2: valorCampo("s-spo2"),
        glicemia: valorCampo("s-glic"),
        peso: valorCampo("s-peso"),

        medidoPor: sessao ? sessao.nome : "Profissional",
        medidoEm: Date.now()
    };

    atendimento.anotacoes = valorCampo("s-notas");

    var riscoAnterior = atendimento.risco;

    if (riscoEscolhido) {
        atendimento.risco = riscoEscolhido;

        if (riscoEscolhido !== riscoAnterior) {
            atendimento.riscoPor = sessao ? sessao.nome : "Profissional";
            atendimento.riscoConfirmado =
                !!sessao && sessao.papel === "enfermeiro";
        } else if (sessao && sessao.papel === "enfermeiro") {
            atendimento.riscoConfirmado = true;
        }

        if (atendimento.status === "aguardando") {
            atendimento.status = "em atendimento";
        }
    }

    if (!Array.isArray(atendimento.eventos)) {
        atendimento.eventos = [];
    }

    atendimento.eventos.push({
        quando: Date.now(),
        texto: "Ficha atualizada"
    });

    salvarBanco();

    registrarAcao(
        "Salvou a ficha do atendimento",
        atendimento.senhaChamada
    );

    mostrarMensagem(
        sessao && sessao.papel === "enfermeiro"
            ? "Ficha salva. Confira os dados antes de finalizar."
            : "Ficha salva. A classificação de risco precisa ser confirmada pelo enfermeiro."
    );

    mostrarFila();
}


// =====================================================
// FINALIZAR ATENDIMENTO
// =====================================================

function finalizarAtendimento(idAtendimento) {
    if (!sessao || sessao.papel !== "enfermeiro") {
        mostrarMensagem(
            "Somente o enfermeiro pode finalizar o atendimento."
        );
        return;
    }

    var atendimento = banco.atendimentos.find(function (item) {
        return item.id === idAtendimento;
    });

    if (!atendimento) return;

    if (!atendimento.risco) {
        mostrarMensagem(
            "Selecione uma classificação de risco antes de finalizar."
        );
        return;
    }

    var confirmar = confirm(
        "Deseja finalizar o atendimento " +
        atendimento.senhaChamada + "?"
    );

    if (!confirmar) return;

    atendimento.status = "finalizado";
    atendimento.riscoConfirmado = true;
    atendimento.finalizadoEm = Date.now();

    if (!Array.isArray(atendimento.eventos)) {
        atendimento.eventos = [];
    }

    atendimento.eventos.push({
        quando: Date.now(),
        texto: "Atendimento finalizado pelo enfermeiro"
    });

    salvarBanco();

    registrarAcao(
        "Finalizou o atendimento",
        atendimento.senhaChamada
    );

    atendimentoSelecionado = null;

    var ficha = pegar("fichaPaciente");
    if (ficha) ficha.innerHTML = "";

    mostrarMensagem("Atendimento finalizado.");
    mostrarFila();
}


// =====================================================
// DADOS DE DEMONSTRAÇÃO
// =====================================================

function criarDadosDeExemplo() {
    carregarBanco();

    var exemplos = [
        {
            nome: "Maria de Exemplo",
            nascimento: "1985-05-12",
            cpf: "11111111111",
            rg: "1234567",
            telefone: "(11) 90000-0001",
            alergias: "Nenhuma informada",
            doencas: ["Hipertensão"],
            motivo: "Dor de cabeça",
            sintomas: "Dor de cabeça iniciada hoje",
            dor: 4
        },
        {
            nome: "João de Exemplo",
            nascimento: "1972-08-20",
            cpf: "22222222222",
            rg: "2345678",
            telefone: "(11) 90000-0002",
            alergias: "Dipirona",
            doencas: ["Diabetes"],
            motivo: "Mal-estar",
            sintomas: "Relata mal-estar",
            dor: 6
        },
        {
            nome: "Ana de Exemplo",
            nascimento: "1998-03-04",
            cpf: "33333333333",
            rg: "3456789",
            telefone: "(11) 90000-0003",
            alergias: "Nenhuma informada",
            doencas: [],
            motivo: "Dor abdominal",
            sintomas: "Dor abdominal iniciada há algumas horas",
            dor: 7
        }
    ];

    var criados = 0;

    exemplos.forEach(function (exemplo, indice) {
        var existe = banco.pacientes.some(function (paciente) {
            return paciente.cpf === exemplo.cpf;
        });

        if (existe) return;

        var paciente = {
            id: criarId(),
            nome: exemplo.nome,
            nomeSocial: "",
            nascimento: exemplo.nascimento,
            cpf: exemplo.cpf,
            rg: exemplo.rg,
            sus: "",
            govbr: "",
            telefone: exemplo.telefone,
            sangue: "",
            endereco: {
                cep: "",
                rua: "",
                numero: "",
                bairro: "",
                cidade: "",
                estado: ""
            },
            alergias: exemplo.alergias,
            doencas: exemplo.doencas,
            outras: "",
            medicamentos: "",
            emergencia: {
                nome: "Contato de exemplo",
                parentesco: "Familiar",
                telefone: exemplo.telefone
            },
            senha: protegerSenha("123456"),
            consentimento: {
                aceito: true,
                quando: Date.now(),
                versao: "1.0"
            },
            criadoEm: Date.now()
        };

        banco.pacientes.push(paciente);

        var numero = banco.atendimentos.length + 1;

        banco.atendimentos.push({
            id: criarId(),
            pacienteId: paciente.id,
            senhaChamada: "T" + String(numero).padStart(3, "0"),
            abertoEm: Date.now() - (indice * 600000),
            motivo: exemplo.motivo,
            inicio: "Hoje",
            sintomas: exemplo.sintomas,
            dor: exemplo.dor,
            risco: null,
            riscoConfirmado: false,
            status: "aguardando",
            acompanhante: {
                nome: "",
                parentesco: "",
                rg: "",
                cpf: "",
                telefone: "",
                endereco: ""
            },
            sinais: {},
            anotacoes: "",
            eventos: [{
                quando: Date.now(),
                texto: "Ficha criada como exemplo"
            }]
        });

        criados++;
    });

    salvarBanco();

    mostrarMensagem(
        criados
            ? criados + " pacientes de exemplo foram criados."
            : "Os pacientes de exemplo já existem."
    );

    if (sessao && sessao.papel !== "paciente") {
        mostrarFila();
    }
}


// O botão de demonstração é opcional.
// Se não existir no index.html, o JavaScript continua funcionando.
var btnDemo = pegar("btnDemo");

if (btnDemo) {
    btnDemo.addEventListener("click", criarDadosDeExemplo);
}


// =====================================================
// INICIALIZAÇÃO DO SITE
// =====================================================

// Restaura o tema salvo.
try {
    var temaSalvo = localStorage.getItem("triagem_tema");

    if (temaSalvo === "dark" || temaSalvo === "light") {
        document.documentElement.setAttribute(
            "data-theme",
            temaSalvo
        );
    }
} catch (erro) {
    console.warn("Não foi possível restaurar o tema.", erro);
}

// Carrega os dados e monta a escala de dor.
carregarBanco();
montarEscalaDor();


// Atualiza as telas quando outra aba do mesmo navegador altera os dados.
window.addEventListener("storage", function () {
    carregarBanco();

    var telaProfissional = pegar("tela-prof-painel");

    if (
        telaProfissional &&
        telaProfissional.classList.contains("ativa")
    ) {
        mostrarFila();
    }

    var telaPaciente = pegar("tela-paciente-painel");

    if (
        telaPaciente &&
        telaPaciente.classList.contains("ativa")
    ) {
        mostrarPainelPaciente();
    }
});


// Atualiza a fila profissional a cada minuto.
setInterval(function () {
    var telaProfissional = pegar("tela-prof-painel");

    if (
        telaProfissional &&
        telaProfissional.classList.contains("ativa")
    ) {
        mostrarFila();
    }
}, 60000);

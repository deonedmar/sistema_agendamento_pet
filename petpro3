import streamlit as st
import sqlite3
import pandas as pd
from datetime import datetime, date
from contextlib import contextmanager

# ══════════════════════════════════════════════════════════════════════════════
#  CONFIGURAÇÃO DA PÁGINA
# ══════════════════════════════════════════════════════════════════════════════
st.set_page_config(
    page_title="Pet & Taxi Pro",
    page_icon="🐾",
    layout="wide",
    initial_sidebar_state="expanded",
)

# ══════════════════════════════════════════════════════════════════════════════
#  ESTILOS
# ══════════════════════════════════════════════════════════════════════════════
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Sans:wght@300;400;500&display=swap');

html, body, [class*="css"] { font-family: 'DM Sans', sans-serif; }
h1,h2,h3 { font-family: 'Syne', sans-serif !important; font-weight: 800; }

[data-testid="stSidebar"] {
    background: #0f172a !important;
    border-right: 1px solid #1e293b;
}
[data-testid="stSidebar"] * { color: #cbd5e1 !important; }
[data-testid="stSidebar"] .stRadio label {
    padding: 10px 14px;
    border-radius: 8px;
    margin-bottom: 4px;
    display: block;
    transition: background .2s;
}
[data-testid="stSidebar"] .stRadio label:hover { background: #1e293b; }

[data-testid="metric-container"] {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 16px;
    padding: 20px !important;
    box-shadow: 0 1px 3px rgba(0,0,0,.06);
}
[data-testid="stMetricValue"] {
    font-family: 'Syne', sans-serif !important;
    font-size: 2rem !important;
}

.stButton > button[kind="primary"] {
    background: #0f172a !important;
    color: white !important;
    border: none !important;
    border-radius: 10px !important;
    font-family: 'Syne', sans-serif !important;
    font-weight: 700 !important;
    padding: 10px 24px !important;
    transition: opacity .2s !important;
}
.stButton > button[kind="primary"]:hover { opacity: .85 !important; }

/* Botão de perigo (excluir) */
.btn-danger > button {
    background: #EF4444 !important;
    color: white !important;
    border: none !important;
    border-radius: 10px !important;
    font-weight: 700 !important;
    transition: opacity .2s !important;
}
.btn-danger > button:hover { opacity: .8 !important; }

.pet-card {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 14px;
    padding: 18px 22px;
    margin-bottom: 12px;
    box-shadow: 0 1px 4px rgba(0,0,0,.05);
    transition: box-shadow .2s;
}
.pet-card:hover { box-shadow: 0 4px 16px rgba(0,0,0,.1); }

.badge {
    display: inline-block;
    padding: 4px 14px;
    border-radius: 20px;
    font-size: .78rem;
    font-weight: 700;
    font-family: 'Syne', sans-serif;
    letter-spacing: .03em;
}

/* Legenda de status */
.legenda-wrap {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    margin: 10px 0 18px 0;
}
.legenda-item {
    display: flex;
    align-items: center;
    gap: 6px;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 5px 12px;
    font-size: .8rem;
}
.legenda-dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    display: inline-block;
    flex-shrink: 0;
}

.info-box {
    background: #f0f9ff;
    border-left: 4px solid #0ea5e9;
    border-radius: 0 10px 10px 0;
    padding: 14px 18px;
    margin: 10px 0;
}
.warn-box {
    background: #fffbeb;
    border-left: 4px solid #f59e0b;
    border-radius: 0 10px 10px 0;
    padding: 14px 18px;
    margin: 10px 0;
}
.preco-card {
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 12px;
    padding: 14px 18px;
    margin-bottom: 6px;
}
.cliente-card {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 14px;
    padding: 16px 20px;
    margin-bottom: 10px;
    box-shadow: 0 1px 3px rgba(0,0,0,.04);
}
.stExpander { border: 1px solid #e2e8f0 !important; border-radius: 12px !important; }
div[data-testid="stForm"] { border: none !important; }
</style>
""", unsafe_allow_html=True)


# ══════════════════════════════════════════════════════════════════════════════
#  BANCO DE DADOS
# ══════════════════════════════════════════════════════════════════════════════
DB = "pet_taxi.db"


@contextmanager
def conn():
    c = sqlite3.connect(DB, check_same_thread=False)
    c.row_factory = sqlite3.Row
    c.execute("PRAGMA journal_mode=WAL")
    try:
        yield c
        c.commit()
    except Exception as e:
        c.rollback()
        raise e
    finally:
        c.close()


def init_db():
    with conn() as c:
        c.executescript("""
        CREATE TABLE IF NOT EXISTS agendamentos (
            id        INTEGER PRIMARY KEY AUTOINCREMENT,
            tutor     TEXT NOT NULL,
            pet       TEXT NOT NULL,
            servico   TEXT NOT NULL,
            data      TEXT NOT NULL,
            horario   TEXT NOT NULL,
            logistica TEXT NOT NULL DEFAULT 'Sem Transporte',
            endereco  TEXT DEFAULT '',
            status    TEXT NOT NULL DEFAULT 'Agendado',
            valor     REAL NOT NULL DEFAULT 0,
            criado_em TEXT DEFAULT (datetime('now','localtime'))
        );
        CREATE TABLE IF NOT EXISTS clientes (
            id              INTEGER PRIMARY KEY AUTOINCREMENT,
            tutor           TEXT NOT NULL,
            pet             TEXT NOT NULL,
            ultimo_endereco TEXT DEFAULT '',
            telefone        TEXT DEFAULT '',
            total_servicos  INTEGER DEFAULT 0,
            criado_em       TEXT DEFAULT (datetime('now','localtime')),
            UNIQUE(tutor, pet)
        );
        CREATE TABLE IF NOT EXISTS historico (
            id        INTEGER PRIMARY KEY AUTOINCREMENT,
            tutor     TEXT NOT NULL,
            pet       TEXT NOT NULL,
            servico   TEXT NOT NULL,
            data      TEXT NOT NULL,
            valor     REAL NOT NULL,
            criado_em TEXT DEFAULT (datetime('now','localtime'))
        );
        CREATE TABLE IF NOT EXISTS precos (
            id    INTEGER PRIMARY KEY AUTOINCREMENT,
            tipo  TEXT NOT NULL,
            nome  TEXT NOT NULL UNIQUE,
            valor REAL NOT NULL,
            descricao TEXT DEFAULT ''
        );
        """)
        # Adicionar coluna telefone se não existir (migração)
        try:
            c.execute("ALTER TABLE clientes ADD COLUMN telefone TEXT DEFAULT ''")
        except Exception:
            pass
        # Adicionar coluna descricao em precos se não existir (migração)
        try:
            c.execute("ALTER TABLE precos ADD COLUMN descricao TEXT DEFAULT ''")
        except Exception:
            pass

        # Inserir preços padrão apenas se a tabela estiver vazia
        qtd = c.execute("SELECT COUNT(*) FROM precos").fetchone()[0]
        if qtd == 0:
            defaults = [
                ("servico",   "Banho",                50.0, "Banho completo com shampoo e secagem"),
                ("servico",   "Tosa",                 65.0, "Tosa higiênica ou padrão de raça"),
                ("servico",   "Banho + Tosa",         110.0, "Banho completo + tosa no mesmo dia"),
                ("servico",   "Consulta Veterinária",  90.0, "Consulta clínica geral com veterinário"),
                ("servico",   "Hidratação",            45.0, "Hidratação profunda do pelo"),
                ("logistica", "Sem Transporte",          0.0, "Cliente leva e busca o pet"),
                ("logistica", "Busca + Entrega",        20.0, "Buscamos e entregamos em casa"),
                ("logistica", "Só Busca",              10.0, "Buscamos o pet na casa do cliente"),
                ("logistica", "Só Entrega",            10.0, "Entregamos o pet após o serviço"),
            ]
            c.executemany(
                "INSERT INTO precos (tipo, nome, valor, descricao) VALUES (?,?,?,?)", defaults
            )


init_db()


# ══════════════════════════════════════════════════════════════════════════════
#  HELPERS DE BANCO
# ══════════════════════════════════════════════════════════════════════════════

def df_query(sql, params=()):
    with conn() as c:
        return pd.read_sql_query(sql, c, params=params)


def executar(sql, params=()):
    with conn() as c:
        cur = c.execute(sql, params)
        return cur.lastrowid


def horario_livre(data_str, horario):
    r = df_query(
        "SELECT COUNT(*) as n FROM agendamentos WHERE data=? AND horario=?",
        (data_str, horario)
    )
    return int(r["n"].iloc[0]) < 3


def upsert_cliente(tutor, pet, endereco):
    with conn() as c:
        c.execute("""
            INSERT INTO clientes (tutor, pet, ultimo_endereco, total_servicos)
            VALUES (?, ?, ?, 1)
            ON CONFLICT(tutor, pet) DO UPDATE SET
                ultimo_endereco = excluded.ultimo_endereco,
                total_servicos  = total_servicos + 1
        """, (tutor, pet, endereco))


# ══════════════════════════════════════════════════════════════════════════════
#  PREÇOS DINÂMICOS
# ══════════════════════════════════════════════════════════════════════════════

def get_precos():
    df = df_query("SELECT nome, valor FROM precos WHERE tipo='servico' ORDER BY id")
    return dict(zip(df["nome"], df["valor"]))


def get_taxas():
    df = df_query("SELECT nome, valor FROM precos WHERE tipo='logistica' ORDER BY id")
    return dict(zip(df["nome"], df["valor"]))


def get_precos_completo():
    return df_query("SELECT * FROM precos WHERE tipo='servico' ORDER BY id")


def get_taxas_completo():
    return df_query("SELECT * FROM precos WHERE tipo='logistica' ORDER BY id")


def salvar_preco(nome, valor, descricao=""):
    executar("UPDATE precos SET valor=?, descricao=? WHERE nome=?", (valor, descricao, nome))


def adicionar_item(tipo, nome, valor, descricao=""):
    with conn() as c:
        c.execute(
            "INSERT OR IGNORE INTO precos (tipo, nome, valor, descricao) VALUES (?,?,?,?)",
            (tipo, nome, valor, descricao)
        )


def remover_preco(nome):
    executar("DELETE FROM precos WHERE nome=?", (nome,))


# ══════════════════════════════════════════════════════════════════════════════
#  CONFIGURAÇÕES FIXAS
# ══════════════════════════════════════════════════════════════════════════════
HORARIOS = [f"{h:02d}:{m:02d}" for h in range(8, 19) for m in (0, 30)]

STATUS_BALCAO = ["Agendado", "Aguardando", "Em Banho", "Secando", "Pronto", "Finalizado"]
STATUS_TAXI   = ["Agendado", "A Caminho", "Pet Coletado", "Na Loja",
                 "Serviço Concluído", "Retornando", "Finalizado"]

COR_STATUS = {
    "Agendado":          ("#6B7280", "#F3F4F6"),
    "Aguardando":        ("#D97706", "#FFFBEB"),
    "Em Banho":          ("#2563EB", "#EFF6FF"),
    "Secando":           ("#7C3AED", "#F5F3FF"),
    "A Caminho":         ("#D97706", "#FFFBEB"),
    "Pet Coletado":      ("#0891B2", "#ECFEFF"),
    "Na Loja":           ("#7C3AED", "#F5F3FF"),
    "Serviço Concluído": ("#059669", "#ECFDF5"),
    "Retornando":        ("#EA580C", "#FFF7ED"),
    "Pronto":            ("#059669", "#ECFDF5"),
    "Finalizado":        ("#1F2937", "#F9FAFB"),
}


def badge(status):
    cor_txt, cor_bg = COR_STATUS.get(status, ("#6B7280", "#F3F4F6"))
    return (
        f"<span class='badge' style='color:{cor_txt};background:{cor_bg}'>"
        f"{status}</span>"
    )


def legenda_status(lista_status):
    """Renderiza legenda visual para uma lista de status."""
    itens = ""
    for s in lista_status:
        cor_txt, cor_bg = COR_STATUS.get(s, ("#6B7280", "#F3F4F6"))
        itens += f"""
        <div class='legenda-item'>
            <span class='legenda-dot' style='background:{cor_txt}'></span>
            <span style='color:#374151'>{s}</span>
        </div>"""
    return f"<div class='legenda-wrap'>{itens}</div>"


# ══════════════════════════════════════════════════════════════════════════════
#  SIDEBAR / NAVEGAÇÃO
# ══════════════════════════════════════════════════════════════════════════════
with st.sidebar:
    st.markdown("""
    <div style='text-align:center;padding:20px 0 10px'>
        <div style='font-size:3rem'>🐾</div>
        <div style='font-family:Syne,sans-serif;font-weight:800;font-size:1.3rem;color:#f1f5f9'>
            Pet & Taxi Pro
        </div>
        <div style='font-size:.78rem;color:#64748b;margin-top:4px'>Sistema de Gestão</div>
    </div>
    """, unsafe_allow_html=True)
    st.markdown("---")

    pagina = st.radio("", [
        "📊  Dashboard",
        "📝  Novo Agendamento",
        "🏠  Atendimento Balcão",
        "🚐  Logística Taxi Dog",
        "👥  Base de Clientes",
        "💲  Preços e Serviços",
    ], label_visibility="collapsed")

    st.markdown("---")
    hoje_fmt = datetime.today().strftime("%d/%m/%Y")
    st.markdown(
        f"<div style='font-size:.8rem;color:#475569;text-align:center'>📅 {hoje_fmt}</div>",
        unsafe_allow_html=True
    )


HOJE = str(date.today())


# ══════════════════════════════════════════════════════════════════════════════
#  1. DASHBOARD
# ══════════════════════════════════════════════════════════════════════════════
if "Dashboard" in pagina:
    st.markdown("## 📊 Painel de Operações")

    df_hoje = df_query(
        "SELECT * FROM agendamentos WHERE data = ? ORDER BY horario", (HOJE,)
    )
    df_mes = df_query(
        "SELECT * FROM agendamentos WHERE data LIKE ? AND status='Finalizado'",
        (HOJE[:7] + "%",)
    )

    c1, c2, c3, c4, c5 = st.columns(5)
    c1.metric("🗓 Agendamentos Hoje", len(df_hoje))
    c2.metric(
        "🚐 Com Transporte",
        len(df_hoje[df_hoje["logistica"] != "Sem Transporte"]) if not df_hoje.empty else 0
    )
    c3.metric(
        "✅ Finalizados Hoje",
        len(df_hoje[df_hoje["status"] == "Finalizado"]) if not df_hoje.empty else 0
    )
    c4.metric(
        "💰 Faturamento Hoje",
        f"R$ {df_hoje['valor'].sum():.2f}" if not df_hoje.empty else "R$ 0,00"
    )
    c5.metric(
        "📈 Faturamento Mês",
        f"R$ {df_mes['valor'].sum():.2f}" if not df_mes.empty else "R$ 0,00"
    )

    st.divider()

    if df_hoje.empty:
        st.markdown("""
        <div class='info-box'>
            🐾 Nenhum agendamento para hoje. Use <b>Novo Agendamento</b> para começar!
        </div>
        """, unsafe_allow_html=True)
    else:
        hora_agora = datetime.now().strftime("%H:%M")
        atrasados  = df_hoje[
            (df_hoje["horario"] < hora_agora) &
            (~df_hoje["status"].isin(["Finalizado"]))
        ]
        if not atrasados.empty:
            st.markdown(
                f"<div class='warn-box'>⚠️ <b>{len(atrasados)} pet(s)</b> "
                f"com horário passado ainda não finalizados!</div>",
                unsafe_allow_html=True
            )

        col_esq, col_dir = st.columns([3, 2])

        with col_esq:
            st.markdown("### 📋 Fila do Dia")
            for _, r in df_hoje.iterrows():
                atrasado = r["horario"] < hora_agora and r["status"] not in ["Finalizado"]
                cor_hora = "#EF4444" if atrasado else "#0f172a"
                st.markdown(f"""
                <div class='pet-card'>
                    <div style='display:flex;justify-content:space-between;align-items:center'>
                        <div>
                            <span style='font-family:Syne,sans-serif;font-size:1.5rem;
                                  font-weight:800;color:{cor_hora}'>{r['horario']}</span>
                            <span style='margin-left:12px;font-weight:600;font-size:1.05rem'>
                                🐾 {r['pet']}</span>
                            <span style='color:#64748b;margin-left:8px'>· {r['tutor']}</span>
                        </div>
                        {badge(r['status'])}
                    </div>
                    <div style='margin-top:8px;color:#475569;font-size:.9rem'>
                        ✂️ {r['servico']} &nbsp;·&nbsp;
                        🚐 {r['logistica']} &nbsp;·&nbsp;
                        💰 R$ {r['valor']:.2f}
                    </div>
                </div>
                """, unsafe_allow_html=True)

        with col_dir:
            st.markdown("### 📊 Serviços de Hoje")
            cont = df_hoje["servico"].value_counts().reset_index()
            cont.columns = ["Serviço", "Qtd"]
            st.bar_chart(cont.set_index("Serviço"), height=200)

            st.markdown("### 🚐 Rotas Ativas")
            taxi_ativo = df_hoje[
                (df_hoje["logistica"] != "Sem Transporte") &
                (~df_hoje["status"].isin(["Finalizado"]))
            ]
            if taxi_ativo.empty:
                st.caption("Nenhuma rota ativa agora.")
            else:
                for _, r in taxi_ativo.iterrows():
                    st.markdown(f"""
                    <div style='background:#f8fafc;border:1px solid #e2e8f0;border-radius:10px;
                                padding:10px 14px;margin-bottom:8px'>
                        <b>{r['pet']}</b> — {r['horario']}<br>
                        <span style='font-size:.85rem;color:#64748b'>{r['logistica']}</span>
                        &nbsp; {badge(r['status'])}
                    </div>
                    """, unsafe_allow_html=True)


# ══════════════════════════════════════════════════════════════════════════════
#  2. NOVO AGENDAMENTO
# ══════════════════════════════════════════════════════════════════════════════
elif "Agendamento" in pagina:
    st.markdown("## 📝 Novo Agendamento")
    st.divider()

    PRECOS = get_precos()
    TAXAS  = get_taxas()

    # Busca rápida de cliente existente
    with st.expander("🔍 Buscar cliente já cadastrado (opcional)"):
        busca_rapida = st.text_input("Digite nome do tutor ou pet para pré-preencher")
        if busca_rapida:
            res = df_query(
                "SELECT * FROM clientes WHERE tutor LIKE ? OR pet LIKE ? LIMIT 5",
                (f"%{busca_rapida}%", f"%{busca_rapida}%")
            )
            if not res.empty:
                for _, cli in res.iterrows():
                    label = f"📋 {cli['tutor']} — {cli['pet']}"
                    if cli["ultimo_endereco"]:
                        label += f"  ({cli['ultimo_endereco']})"
                    if st.button(label, key=f"sel_{cli['id']}"):
                        st.session_state["pf_tutor"]    = cli["tutor"]
                        st.session_state["pf_pet"]      = cli["pet"]
                        st.session_state["pf_endereco"] = cli["ultimo_endereco"]
                        st.rerun()
            else:
                st.caption("Nenhum cliente encontrado.")

    def ss(key, default=""):
        return st.session_state.get(key, default)

    with st.form("form_agendamento", clear_on_submit=False):
        st.markdown("#### 👤 Dados do Cliente")
        c1, c2 = st.columns(2)
        tutor = c1.text_input("Nome do Tutor *", value=ss("pf_tutor"))
        pet   = c2.text_input("Nome do Pet *",   value=ss("pf_pet"))

        st.markdown("#### ✂️ Serviço")
        c3, c4, c5 = st.columns(3)
        servico = c3.selectbox("Serviço *", list(PRECOS.keys()))
        data    = c4.date_input("Data *", min_value=date.today())
        horario = c5.selectbox("Horário *", HORARIOS)

        st.markdown("#### 🚐 Transporte")
        logistica = st.radio(
            "Configuração de Transporte", list(TAXAS.keys()), horizontal=True
        )
        endereco = st.text_input(
            "Endereço Completo", value=ss("pf_endereco"),
            placeholder="Rua, número, bairro, cidade",
            disabled=(logistica == "Sem Transporte")
        )

        valor = PRECOS.get(servico, 0) + TAXAS.get(logistica, 0)
        taxa  = TAXAS.get(logistica, 0)
        st.markdown(f"""
        <div style='background:#f0fdf4;border:1px solid #bbf7d0;border-radius:10px;
                    padding:14px 20px;margin:10px 0'>
            💰 <b>Valor estimado:</b>
            <span style='font-family:Syne,sans-serif;font-size:1.3rem;font-weight:800;color:#15803d'>
                R$ {valor:.2f}
            </span>
            <span style='color:#64748b;font-size:.85rem'>
                &nbsp;(Serviço: R$ {PRECOS.get(servico, 0):.2f}
                {f" + Transporte: R$ {taxa:.2f}" if taxa > 0 else ""})
            </span>
        </div>
        """, unsafe_allow_html=True)

        st.markdown("#### 📝 Observações")
        st.text_area(
            "Observações (opcional)", height=70,
            placeholder="Comportamento, raça, alergias..."
        )

        submitted = st.form_submit_button(
            "✅ Confirmar Agendamento", type="primary", use_container_width=True
        )

    if submitted:
        erros = []
        tutor = tutor.strip().title()
        pet   = pet.strip().title()
        if not tutor: erros.append("Nome do tutor é obrigatório.")
        if not pet:   erros.append("Nome do pet é obrigatório.")
        if logistica != "Sem Transporte" and not endereco.strip():
            erros.append("Endereço obrigatório para serviço com transporte.")
        data_str = str(data)
        if not horario_livre(data_str, horario):
            erros.append(f"Horário {horario} está cheio para esta data (máx. 3 pets).")

        if erros:
            for e in erros:
                st.error(f"❌ {e}")
        else:
            novo_id = executar(
                """INSERT INTO agendamentos
                   (tutor, pet, servico, data, horario, logistica, endereco, status, valor)
                   VALUES (?,?,?,?,?,?,?,'Agendado',?)""",
                (tutor, pet, servico, data_str, horario, logistica, endereco.strip(), valor)
            )
            upsert_cliente(tutor, pet, endereco.strip())
            executar(
                "INSERT INTO historico (tutor, pet, servico, data, valor) VALUES (?,?,?,?,?)",
                (tutor, pet, servico, data_str, valor)
            )
            for k in ["pf_tutor", "pf_pet", "pf_endereco"]:
                st.session_state.pop(k, None)

            st.success(
                f"✅ Agendamento #{novo_id} criado! "
                f"{pet} agendado para {data.strftime('%d/%m/%Y')} às {horario}."
            )
            st.balloons()


# ══════════════════════════════════════════════════════════════════════════════
#  3. ATENDIMENTO BALCÃO
# ══════════════════════════════════════════════════════════════════════════════
elif "Balcão" in pagina:
    st.markdown("## 🏠 Atendimento Balcão")

    # ── Legenda de status ─────────────────────────────────────────────────────
    st.markdown("**Legenda de Status:**")
    st.markdown(legenda_status(STATUS_BALCAO), unsafe_allow_html=True)
    st.divider()

    data_sel = st.date_input("📅 Ver agenda do dia", value=date.today())
    data_str = str(data_sel)

    df_all    = df_query(
        "SELECT * FROM agendamentos WHERE data=? ORDER BY horario", (data_str,)
    )
    df_balcao = df_all[df_all["logistica"] == "Sem Transporte"]

    if df_balcao.empty:
        st.markdown(
            f"<div class='info-box'>Nenhum pet de balcão para "
            f"{data_sel.strftime('%d/%m/%Y')}.</div>",
            unsafe_allow_html=True
        )
    else:
        total       = len(df_balcao)
        finalizados = len(df_balcao[df_balcao["status"] == "Finalizado"])
        st.progress(
            finalizados / total if total > 0 else 0,
            text=f"✅ {finalizados} de {total} pets finalizados"
        )
        st.divider()

        for _, r in df_balcao.iterrows():
            with st.container():
                c1, c2, c3 = st.columns([3, 3, 2])
                with c1:
                    st.markdown(f"""
                    <div style='margin-bottom:4px'>
                        <span style='font-family:Syne,sans-serif;font-size:1.4rem;font-weight:800'>
                            {r['horario']}
                        </span>
                        &nbsp; <b>{r['pet']}</b>
                        <span style='color:#94a3b8'> · {r['tutor']}</span>
                    </div>
                    <div style='color:#475569;font-size:.9rem'>
                        ✂️ {r['servico']} &nbsp;·&nbsp; 💰 R$ {r['valor']:.2f}
                    </div>
                    """, unsafe_allow_html=True)
                with c2:
                    st.markdown(badge(r["status"]), unsafe_allow_html=True)
                    idx  = STATUS_BALCAO.index(r["status"]) if r["status"] in STATUS_BALCAO else 0
                    novo = st.selectbox(
                        "Status", STATUS_BALCAO, index=idx,
                        key=f"bal_{r['id']}", label_visibility="collapsed"
                    )
                with c3:
                    if st.button("💾 Salvar", key=f"save_bal_{r['id']}", type="primary"):
                        executar(
                            "UPDATE agendamentos SET status=? WHERE id=?",
                            (novo, int(r["id"]))
                        )
                        st.success("Atualizado!")
                        st.rerun()
                st.divider()


# ══════════════════════════════════════════════════════════════════════════════
#  4. LOGÍSTICA TAXI DOG
# ══════════════════════════════════════════════════════════════════════════════
elif "Taxi" in pagina:
    st.markdown("## 🚐 Logística Taxi Dog")

    # ── Legenda de status ─────────────────────────────────────────────────────
    st.markdown("**Legenda de Status:**")
    st.markdown(legenda_status(STATUS_TAXI), unsafe_allow_html=True)
    st.divider()

    data_sel = st.date_input("📅 Ver rotas do dia", value=date.today())
    data_str = str(data_sel)

    df_taxi = df_query(
        "SELECT * FROM agendamentos WHERE data=? AND logistica != 'Sem Transporte' ORDER BY horario",
        (data_str,)
    )

    if df_taxi.empty:
        st.markdown(
            f"<div class='info-box'>Sem rotas de Taxi Dog para "
            f"{data_sel.strftime('%d/%m/%Y')}.</div>",
            unsafe_allow_html=True
        )
    else:
        c1, c2, c3, c4 = st.columns(4)
        em_rota   = len(df_taxi[df_taxi["status"].isin(["A Caminho", "Retornando"])])
        na_loja   = len(df_taxi[df_taxi["status"].isin(
            ["Na Loja", "Pet Coletado", "Serviço Concluído"]
        )])
        concluido = len(df_taxi[df_taxi["status"] == "Finalizado"])
        c1.metric("Total de Rotas", len(df_taxi))
        c2.metric("🚗 Em Trânsito",  em_rota)
        c3.metric("🏪 Na Loja",      na_loja)
        c4.metric("✅ Concluídos",   concluido)
        st.divider()

        for _, r in df_taxi.iterrows():
            finalizado = r["status"] == "Finalizado"
            icon = "✅" if finalizado else "📍"
            with st.expander(
                f"{icon}  {r['horario']}  —  {r['pet']}  ({r['logistica']})",
                expanded=not finalizado
            ):
                col_info, col_acao = st.columns([3, 2])

                with col_info:
                    st.markdown(f"""
                    <div class='pet-card'>
                        <div style='font-size:1.1rem;font-weight:700;margin-bottom:8px'>
                            🐾 {r['pet']}
                            <span style='color:#94a3b8;font-weight:400'>· {r['tutor']}</span>
                        </div>
                        <div style='color:#475569;line-height:1.9;font-size:.92rem'>
                            ✂️ <b>Serviço:</b> {r['servico']}<br>
                            🚐 <b>Transporte:</b> {r['logistica']}<br>
                            📍 <b>Endereço:</b> {r['endereco'] or '—'}<br>
                            💰 <b>Valor:</b> R$ {r['valor']:.2f}
                        </div>
                        <div style='margin-top:12px'>{badge(r['status'])}</div>
                    </div>
                    """, unsafe_allow_html=True)

                    if r["endereco"]:
                        end_enc = str(r["endereco"]).replace(" ", "+")
                        bc1, bc2 = st.columns(2)
                        bc1.link_button(
                            "🗺️ Google Maps",
                            f"https://www.google.com/maps/search/?api=1&query={end_enc}",
                            use_container_width=True
                        )
                        bc2.link_button(
                            "🔵 Waze",
                            f"https://waze.com/ul?q={end_enc}",
                            use_container_width=True
                        )

                with col_acao:
                    st.markdown("**Atualizar progresso:**")
                    idx = STATUS_TAXI.index(r["status"]) if r["status"] in STATUS_TAXI else 0
                    novo_status = st.select_slider(
                        "Status",
                        options=STATUS_TAXI,
                        value=STATUS_TAXI[idx],
                        key=f"taxi_{r['id']}",
                        label_visibility="collapsed"
                    )
                    if st.button(
                        "💾 Salvar Status", key=f"save_taxi_{r['id']}",
                        type="primary", use_container_width=True
                    ):
                        executar(
                            "UPDATE agendamentos SET status=? WHERE id=?",
                            (novo_status, int(r["id"]))
                        )
                        st.success("Status atualizado!")
                        st.rerun()


# ══════════════════════════════════════════════════════════════════════════════
#  5. BASE DE CLIENTES
# ══════════════════════════════════════════════════════════════════════════════
elif "Clientes" in pagina:
    st.markdown("## 👥 Base de Clientes")
    st.divider()

    aba1, aba2 = st.tabs(["📋 Clientes Cadastrados", "➕ Cadastrar Manualmente"])

    # ── ABA 1: Listar, Editar e Excluir ──────────────────────────────────────
    with aba1:
        col_b, col_o = st.columns([3, 1])
        busca = col_b.text_input(
            "🔍 Buscar por nome do tutor ou pet", placeholder="Digite para filtrar..."
        )
        ordem = col_o.selectbox("Ordenar por", ["Tutor A-Z", "Mais Serviços"])

        if busca.strip():
            df_cli = df_query(
                "SELECT * FROM clientes WHERE tutor LIKE ? OR pet LIKE ?",
                (f"%{busca.strip()}%", f"%{busca.strip()}%")
            )
        else:
            df_cli = df_query("SELECT * FROM clientes")

        df_cli = df_cli.sort_values(
            "total_servicos" if ordem == "Mais Serviços" else "tutor",
            ascending=(ordem != "Mais Serviços")
        )

        if df_cli.empty:
            st.info("Nenhum cliente encontrado.")
        else:
            cc1, cc2 = st.columns(2)
            cc1.metric("Total de Clientes", len(df_cli))
            cc2.metric("Total de Serviços Realizados", int(df_cli["total_servicos"].sum()))
            st.divider()

            for _, cli in df_cli.iterrows():
                cid = int(cli["id"])
                with st.container():
                    # ── Cabeçalho do card ─────────────────────────────────────
                    st.markdown(f"""
                    <div class='cliente-card'>
                        <div style='display:flex;justify-content:space-between;align-items:flex-start'>
                            <div>
                                <span style='font-size:1.1rem;font-weight:700'>
                                    🐾 {cli['pet']}
                                </span>
                                <span style='color:#64748b;margin-left:8px'>· {cli['tutor']}</span>
                                <br>
                                <span style='font-size:.85rem;color:#64748b'>
                                    📍 {cli['ultimo_endereco'] or '—'}
                                    &nbsp;·&nbsp;
                                    📞 {cli.get('telefone','') or '—'}
                                    &nbsp;·&nbsp;
                                    🔁 <b>{cli['total_servicos']}</b> serviços
                                </span>
                            </div>
                        </div>
                    </div>
                    """, unsafe_allow_html=True)

                    col_hist, col_edit, col_del = st.columns([3, 2, 1])

                    # ── Histórico ─────────────────────────────────────────────
                    with col_hist:
                        with st.expander(f"📜 Histórico de {cli['pet']}"):
                            hist = df_query(
                                "SELECT servico, data, valor FROM historico "
                                "WHERE tutor=? AND pet=? ORDER BY data DESC",
                                (cli["tutor"], cli["pet"])
                            )
                            if hist.empty:
                                st.caption("Nenhum serviço registrado no histórico.")
                            else:
                                st.markdown(
                                    f"💰 **Total gasto:** R$ {hist['valor'].sum():.2f}"
                                )
                                st.dataframe(
                                    hist.rename(columns={
                                        "servico": "Serviço",
                                        "data":    "Data",
                                        "valor":   "Valor (R$)"
                                    }),
                                    use_container_width=True, hide_index=True
                                )

                    # ── Editar ────────────────────────────────────────────────
                    with col_edit:
                        with st.expander("✏️ Editar dados"):
                            with st.form(f"form_edit_{cid}", clear_on_submit=False):
                                e_tutor = st.text_input(
                                    "Nome do Tutor", value=cli["tutor"],
                                    key=f"et_{cid}"
                                )
                                e_pet = st.text_input(
                                    "Nome do Pet", value=cli["pet"],
                                    key=f"ep_{cid}"
                                )
                                e_end = st.text_input(
                                    "Endereço", value=cli["ultimo_endereco"] or "",
                                    key=f"ee_{cid}"
                                )
                                e_tel = st.text_input(
                                    "Telefone / WhatsApp",
                                    value=cli.get("telefone", "") or "",
                                    key=f"etel_{cid}"
                                )
                                if st.form_submit_button(
                                    "💾 Salvar Alterações", type="primary",
                                    use_container_width=True
                                ):
                                    e_tutor = e_tutor.strip().title()
                                    e_pet   = e_pet.strip().title()
                                    if e_tutor and e_pet:
                                        try:
                                            executar(
                                                """UPDATE clientes
                                                   SET tutor=?, pet=?,
                                                       ultimo_endereco=?, telefone=?
                                                   WHERE id=?""",
                                                (e_tutor, e_pet, e_end.strip(),
                                                 e_tel.strip(), cid)
                                            )
                                            st.success("✅ Dados atualizados!")
                                            st.rerun()
                                        except sqlite3.IntegrityError:
                                            st.error(
                                                "⚠️ Já existe um cadastro com esse "
                                                "tutor e pet."
                                            )
                                    else:
                                        st.error("Tutor e Pet são obrigatórios.")

                    # ── Excluir ───────────────────────────────────────────────
                    with col_del:
                        with st.expander("🗑️ Excluir"):
                            st.warning(
                                f"Excluir **{cli['pet']}** ({cli['tutor']}) e todo "
                                f"o histórico?\n\n**Esta ação não pode ser desfeita.**"
                            )
                            confirmar_key = f"confirm_del_{cid}"
                            st.checkbox(
                                "Confirmo a exclusão", key=confirmar_key
                            )
                            if st.button(
                                "🗑️ Excluir definitivamente",
                                key=f"del_cli_{cid}",
                                type="primary"
                            ):
                                if st.session_state.get(confirmar_key, False):
                                    executar(
                                        "DELETE FROM clientes WHERE id=?", (cid,)
                                    )
                                    executar(
                                        "DELETE FROM historico WHERE tutor=? AND pet=?",
                                        (cli["tutor"], cli["pet"])
                                    )
                                    st.success(
                                        f"✅ {cli['pet']} ({cli['tutor']}) excluído."
                                    )
                                    st.rerun()
                                else:
                                    st.error(
                                        "Marque a caixa de confirmação antes de excluir."
                                    )

                    st.divider()

    # ── ABA 2: Cadastro Manual ────────────────────────────────────────────────
    with aba2:
        with st.form("form_cliente_manual", clear_on_submit=True):
            st.markdown("#### Dados do Cliente")
            mc1, mc2 = st.columns(2)
            n_tutor = mc1.text_input("Nome do Tutor *")
            n_pet   = mc2.text_input("Nome do Pet *")
            n_end   = st.text_input("Endereço Padrão")
            n_tel   = st.text_input("Telefone / WhatsApp")

            if st.form_submit_button("✅ Cadastrar Cliente", type="primary"):
                n_tutor = n_tutor.strip().title()
                n_pet   = n_pet.strip().title()
                if n_tutor and n_pet:
                    try:
                        executar(
                            """INSERT INTO clientes
                               (tutor, pet, ultimo_endereco, telefone, total_servicos)
                               VALUES (?,?,?,?,0)""",
                            (n_tutor, n_pet, n_end.strip(), n_tel.strip())
                        )
                        st.success(f"✅ {n_pet} ({n_tutor}) cadastrado com sucesso!")
                        st.rerun()
                    except sqlite3.IntegrityError:
                        st.warning("⚠️ Este pet já está cadastrado para este tutor.")
                else:
                    st.error("Preencha nome do tutor e do pet.")


# ══════════════════════════════════════════════════════════════════════════════
#  6. PREÇOS E SERVIÇOS
# ══════════════════════════════════════════════════════════════════════════════
elif "Preços" in pagina:
    st.markdown("## 💲 Preços e Serviços")
    st.caption("Alterações aqui refletem imediatamente nos novos agendamentos.")
    st.divider()

    df_srv_full = get_precos_completo()
    df_tax_full = get_taxas_completo()
    PRECOS      = dict(zip(df_srv_full["nome"], df_srv_full["valor"]))
    TAXAS       = dict(zip(df_tax_full["nome"], df_tax_full["valor"]))

    aba_srv, aba_tax, aba_novo = st.tabs([
        "✂️ Serviços",
        "🚐 Taxas de Transporte",
        "➕ Adicionar Novo",
    ])

    # ── Aba: Serviços ─────────────────────────────────────────────────────────
    with aba_srv:
        st.markdown("### Tabela de Serviços")
        st.caption("Edite o valor e a descrição (legenda) de cada serviço e clique em 💾.")
        st.divider()

        for _, row in df_srv_full.iterrows():
            nome        = row["nome"]
            preco_atual = float(row["valor"])
            desc_atual  = str(row["descricao"]) if row["descricao"] else ""

            with st.container():
                st.markdown(f"""
                <div class='preco-card'>
                    <span style='font-weight:700;font-size:1rem'>✂️ {nome}</span>
                    &nbsp;&nbsp;
                    <span style='color:#64748b;font-size:.85rem'>Atual: R$ {preco_atual:.2f}</span>
                    {"<br><span style='font-size:.82rem;color:#94a3b8;font-style:italic'>" + desc_atual + "</span>" if desc_atual else ""}
                </div>
                """, unsafe_allow_html=True)

                ec1, ec2, ec3 = st.columns([2, 3, 1])
                novo_preco = ec1.number_input(
                    "Valor (R$)", min_value=0.0, value=preco_atual,
                    step=5.0, format="%.2f",
                    key=f"srv_val_{nome}", label_visibility="visible"
                )
                nova_desc = ec2.text_input(
                    "Descrição / Legenda",
                    value=desc_atual,
                    placeholder="Breve descrição do serviço...",
                    key=f"srv_desc_{nome}"
                )
                ec3.markdown("<div style='margin-top:28px'></div>", unsafe_allow_html=True)
                if ec3.button("💾", key=f"save_srv_{nome}", help=f"Salvar {nome}"):
                    salvar_preco(nome, novo_preco, nova_desc)
                    st.success(f"✅ {nome} → R$ {novo_preco:.2f}")
                    st.rerun()

                st.divider()

        with st.expander("🗑️ Remover um serviço"):
            srv_rem = st.selectbox("Selecione o serviço", list(PRECOS.keys()), key="rem_srv")
            if st.button("🗑️ Confirmar Remoção", type="primary", key="btn_rem_srv"):
                if len(PRECOS) <= 1:
                    st.error("Deve existir ao menos 1 serviço cadastrado.")
                else:
                    remover_preco(srv_rem)
                    st.success(f"Serviço '{srv_rem}' removido.")
                    st.rerun()

    # ── Aba: Taxas de Transporte ──────────────────────────────────────────────
    with aba_tax:
        st.markdown("### Taxas de Transporte")
        st.caption("A taxa é somada ao preço do serviço. Edite o valor e a descrição.")
        st.divider()

        for _, row in df_tax_full.iterrows():
            nome       = row["nome"]
            taxa_atual = float(row["valor"])
            desc_atual = str(row["descricao"]) if row["descricao"] else ""

            with st.container():
                st.markdown(f"""
                <div class='preco-card'>
                    <span style='font-weight:700;font-size:1rem'>🚐 {nome}</span>
                    &nbsp;&nbsp;
                    <span style='color:#64748b;font-size:.85rem'>Atual: R$ {taxa_atual:.2f}</span>
                    {"<br><span style='font-size:.82rem;color:#94a3b8;font-style:italic'>" + desc_atual + "</span>" if desc_atual else ""}
                </div>
                """, unsafe_allow_html=True)

                tc1, tc2, tc3 = st.columns([2, 3, 1])
                nova_taxa = tc1.number_input(
                    "Taxa (R$)", min_value=0.0, value=taxa_atual,
                    step=5.0, format="%.2f",
                    key=f"tax_val_{nome}", label_visibility="visible"
                )
                nova_desc = tc2.text_input(
                    "Descrição / Legenda",
                    value=desc_atual,
                    placeholder="Ex: Buscamos e entregamos em casa...",
                    key=f"tax_desc_{nome}"
                )
                tc3.markdown("<div style='margin-top:28px'></div>", unsafe_allow_html=True)
                if tc3.button("💾", key=f"save_tax_{nome}", help=f"Salvar {nome}"):
                    salvar_preco(nome, nova_taxa, nova_desc)
                    st.success(f"✅ {nome} → R$ {nova_taxa:.2f}")
                    st.rerun()

                st.divider()

        with st.expander("🗑️ Remover uma opção de transporte"):
            tax_rem = st.selectbox("Selecione a opção", list(TAXAS.keys()), key="rem_tax")
            if st.button("🗑️ Confirmar Remoção", type="primary", key="btn_rem_tax"):
                if tax_rem == "Sem Transporte":
                    st.error("A opção 'Sem Transporte' não pode ser removida.")
                elif len(TAXAS) <= 1:
                    st.error("Deve existir ao menos 1 opção de transporte.")
                else:
                    remover_preco(tax_rem)
                    st.success(f"Opção '{tax_rem}' removida.")
                    st.rerun()

    # ── Aba: Adicionar Novo ───────────────────────────────────────────────────
    with aba_novo:
        st.markdown("### Adicionar Novo Item")
        st.divider()

        tipo_novo = st.radio(
            "Tipo de item",
            ["✂️ Novo Serviço", "🚐 Nova Opção de Transporte"],
            horizontal=True
        )

        with st.form("form_novo_item", clear_on_submit=True):
            nn1, nn2 = st.columns(2)
            novo_nome  = nn1.text_input("Nome *", placeholder="Ex: Tosa Completa, Express...")
            novo_valor = nn2.number_input(
                "Valor (R$) *", min_value=0.0, step=5.0, format="%.2f"
            )
            nova_desc_novo = st.text_input(
                "Descrição / Legenda",
                placeholder="Breve descrição exibida para a equipe..."
            )

            if st.form_submit_button("✅ Adicionar", type="primary"):
                nome_fmt = novo_nome.strip().title()
                if not nome_fmt:
                    st.error("Digite um nome para o item.")
                else:
                    tipo_db = "servico" if "Serviço" in tipo_novo else "logistica"
                    adicionar_item(tipo_db, nome_fmt, novo_valor, nova_desc_novo.strip())
                    st.success(
                        f"✅ '{nome_fmt}' adicionado com valor R$ {novo_valor:.2f}!"
                    )
                    st.rerun()

        st.divider()
        st.markdown("### 📋 Tabela de Preços Atual")
        col_s, col_t = st.columns(2)
        with col_s:
            st.markdown("**✂️ Serviços**")
            df_show = df_srv_full[["nome", "valor", "descricao"]].rename(columns={
                "nome": "Serviço", "valor": "Valor (R$)", "descricao": "Descrição"
            })
            st.dataframe(df_show, use_container_width=True, hide_index=True)
        with col_t:
            st.markdown("**🚐 Transportes**")
            df_show2 = df_tax_full[["nome", "valor", "descricao"]].rename(columns={
                "nome": "Opção", "valor": "Taxa (R$)", "descricao": "Descrição"
            })
            st.dataframe(df_show2, use_container_width=True, hide_index=True)

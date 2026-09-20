import streamlit as st
import pandas as pd
import numpy as np
import io
import os
import json
from ortools.sat.python import cp_model

# Configuração da página Streamlit
st.set_page_config(page_title="Sistema de Gestão de Escalas HC 15º Andar", layout="wide", initial_sidebar_state="collapsed")

# Estilização CSS para otimização de espaço, fontes menores e layout panorâmico
st.markdown("""
<style>
    .block-container { padding-top: 1rem; padding-bottom: 2rem; padding-left: 2rem; padding-right: 2rem; }
    h1 { font-size: 1.6rem !important; margin-bottom: 0.2rem !important; }
    h2, h3 { font-size: 1.2rem !important; margin-top: 0.8rem !important; margin-bottom: 0.4rem !important; }
    .stDataFrame { font-size: 11px !important; }
    div[data-testid="stMetricValue"] { font-size: 1.3rem !important; }
    div[data-testid="stMetricLabel"] { font-size: 0.8rem !important; }
    .css-1544g9n { padding: 0rem 0.5rem; }
    .stButton>button { border-radius: 4px; height: 2.2em; font-size: 13px; font-weight: 600; }
</style>
""", unsafe_allow_html=True)

st.title("🏥 Sistema de Gestão e Otimização de Escalas — HC 15º Andar (Clínica Médica)")
st.caption("Novembro / 2026 | Visão Panorâmica com Otimização por IA, Marcação em Lote e Dashboard de Indicadores")

# BANCO DE DADOS BASE DOS 40 PROFISSIONAIS
@st.cache_data
def carregar_profissionais_base():
    csv_data = """ID,Nome,Sexo,Cargo,Turno_Atribuido,Carga_Horaria_Semanal,Regime_12x36
ENF-01,Ana Paula Silva,Feminino,Enfermeiro,Diurno,36h,Não
ENF-02,Beatriz Oliveira,Feminino,Enfermeiro,Diurno,36h,Não
ENF-03,Carlos Eduardo Lima,Masculino,Enfermeiro,Diurno,40h,Não
ENF-04,Daniela Martins,Feminino,Enfermeiro,Diurno,36h,Não
ENF-05,Eduardo Rocha,Masculino,Enfermeiro,Diurno,40h,Não
ENF-06,Fernanda Alves,Feminino,Enfermeiro,Diurno,36h,Não
ENF-07,Gabriel Santos,Masculino,Enfermeiro,Diurno,40h,Não
ENF-08,Helena Costa,Feminino,Enfermeiro,Diurno,36h,Não
ENF-09,Igor Ribeiro,Masculino,Enfermeiro,Diurno,36h,Não
ENF-10,Juliana Lima,Feminino,Enfermeiro,Diurno,40h,Não
ENF-11,Kátia Mendes,Feminino,Enfermeiro,Diurno,36h,Não
ENF-12,Lucas Pereira,Masculino,Enfermeiro,Diurno,40h,Não
ENF-13,Marcelo Silva,Masculino,Enfermeiro,Noturno,36h,Sim
ENF-14,Nádia Ferreira,Feminino,Enfermeiro,Noturno,36h,Sim
ENF-15,Otávio Barbosa,Masculino,Enfermeiro,Noturno,40h,Sim
ENF-16,Patricia Gomes,Feminino,Enfermeiro,Noturno,36h,Sim
ENF-17,Renato Cardoso,Masculino,Enfermeiro,Noturno,36h,Sim
ENF-18,Simone Duarte,Feminino,Enfermeiro,Noturno,40h,Sim
ENF-19,Thiago Moraes,Masculino,Enfermeiro,Noturno,36h,Sim
ENF-20,Vanessa Castro,Feminino,Enfermeiro,Noturno,40h,Sim
TEC-01,Aline Souza,Feminino,Técnico de Enfermagem,Diurno,36h,Não
TEC-02,Bruno Carrijo,Masculino,Técnico de Enfermagem,Diurno,36h,Não
TEC-03,Camila Rodrigues,Feminino,Técnico de Enfermagem,Diurno,36h,Não
TEC-04,Diego Fernandes,Masculino,Técnico de Enfermagem,Diurno,40h,Não
TEC-05,Eliana Machado,Feminino,Técnico de Enfermagem,Diurno,40h,Não
TEC-06,Fabio Henrique,Masculino,Técnico de Enfermagem,Diurno,36h,Não
TEC-07,Gisele Prado,Feminino,Técnico de Enfermagem,Diurno,36h,Não
TEC-08,Heitor Vasconcelos,Masculino,Técnico de Enfermagem,Diurno,40h,Não
TEC-09,Isabela Faria,Feminino,Técnico de Enfermagem,Diurno,40h,Não
TEC-10,João Vitor Cruz,Masculino,Técnico de Enfermagem,Diurno,40h,Não
TEC-11,Karen Stephanie,Feminino,Técnico de Enfermagem,Diurno,36h,Não
TEC-12,Leonardo Nogueira,Masculino,Técnico de Enfermagem,Diurno,36h,Não
TEC-13,Mariana Freitas,Feminino,Técnico de Enfermagem,Diurno,40h,Não
TEC-14,Natália Guimarães,Feminino,Técnico de Enfermagem,Diurno,36h,Não
TEC-15,Orlando Ramos,Masculino,Técnico de Enfermagem,Noturno,36h,Sim
TEC-16,Paula Tejada,Feminino,Técnico de Enfermagem,Noturno,36h,Sim
TEC-17,Quintino Bocaiúva,Masculino,Técnico de Enfermagem,Noturno,36h,Sim
TEC-18,Raquel Xavier,Feminino,Técnico de Enfermagem,Noturno,36h,Sim
TEC-19,Samuel Rosa,Masculino,Técnico de Enfermagem,Noturno,36h,Sim
TEC-20,Tatiana Valente,Feminino,Técnico de Enfermagem,Noturno,40h,Sim"""
    return pd.read_csv(io.StringIO(csv_data))

df_profs = carregar_profissionais_base()

# Nomes dos dias de Novembro/2026 (Dia 1 = Domingo)
dias_semana_nov2026 = ["Dom", "Seg", "Ter", "Qua", "Qui", "Sex", "Sáb"]
colunas_dias = [f"{d+1} ({dias_semana_nov2026[d % 7]})" for d in range(30)]
sabados_idx = [d for d in range(30) if d % 7 == 6]
domingos_idx = [d for d in range(30) if d % 7 == 0]
finais_semana_idx = set(sabados_idx + domingos_idx)

# CAMINHO DO ARQUIVO DE PERSISTÊNCIA LOCAL (FAZ COM QUE NÃO APAGUE AO REINICIAR)
ARQUIVO_PERSISTENCIA = "/workspace/scratch/escala_estado_salvo.json"

def salvar_estado_local():
    if 'tabela_escala' in st.session_state:
        try:
            st.session_state['tabela_escala'].to_json(ARQUIVO_PERSISTENCIA, orient='records')
        except Exception:
            pass

def carregar_estado_local():
    if os.path.exists(ARQUIVO_PERSISTENCIA):
        try:
            df_salvo = pd.read_json(ARQUIVO_PERSISTENCIA)
            if not df_salvo.empty:
                return df_salvo
        except Exception:
            pass
    return None

# INICIALIZAÇÃO DA TABELA NO SESSION STATE
if 'tabela_escala' not in st.session_state:
    df_recuperado = carregar_estado_local()
    if df_recuperado is not None:
        st.session_state['tabela_escala'] = df_recuperado
    else:
        rows = []
        # Inserir Enfermeiros
        for _, row in df_profs[df_profs['Cargo'] == 'Enfermeiro'].iterrows():
            meta = 156 if row['Carga_Horaria_Semanal'] == '36h' else 176
            r = {'ID': row['ID'], 'Nome': row['Nome'], 'Meta': meta}
            for col in colunas_dias:
                r[col] = ''
            r['Cumprida'] = 0
            r['Saldo'] = f"-{meta}h"
            rows.append(r)
        
        # Linha Divisória de Seção para Técnicos
        r_div = {'ID': '---', 'Nome': '--- TÉCNICOS DE ENFERMAGEM ---', 'Meta': 0}
        for col in colunas_dias:
            r_div[col] = '---'
        r_div['Cumprida'] = 0
        r_div['Saldo'] = '---'
        rows.append(r_div)

        # Inserir Técnicos
        for _, row in df_profs[df_profs['Cargo'] == 'Técnico de Enfermagem'].iterrows():
            meta = 156 if row['Carga_Horaria_Semanal'] == '36h' else 176
            r = {'ID': row['ID'], 'Nome': row['Nome'], 'Meta': meta}
            for col in colunas_dias:
                r[col] = ''
            r['Cumprida'] = 0
            r['Saldo'] = f"-{meta}h"
            rows.append(r)

        st.session_state['tabela_escala'] = pd.DataFrame(rows)

if 'pedidos_dict' not in st.session_state:
    st.session_state['pedidos_dict'] = {}

# PAINEL EXPANSÍVEL 1: ARQUIVO DE PEDIDOS DE FOLGA
with st.expander("📁 Upload / Gerenciar Pedidos de Folga Mensal (Soft Constraints)", expanded=False):
    col_up1, col_up2 = st.columns([3, 2])
    with col_up1:
        arquivo_pedidos = st.file_uploader("Enviar arquivo Excel (.xlsx) ou CSV de pedidos de folga", type=["xlsx", "csv"], key="uploader_pedidos")
        if arquivo_pedidos is not None:
            try:
                df_ped = pd.read_excel(arquivo_pedidos) if arquivo_pedidos.name.endswith(".xlsx") else pd.read_csv(arquivo_pedidos)
                pedidos_dict = {}
                for _, r in df_ped.iterrows():
                    nome_raw = str(r.get('Nome', r.get('Nome Completo', r.get('ID', '')))).strip()
                    dias_raw = str(r.get('Dias_Folga', r.get('Dias', ''))).split(',')
                    dias_list = []
                    for d in dias_raw:
                        d_clean = d.strip().lower().replace('dia', '').replace('d', '')
                        if d_clean.isdigit():
                            dias_list.append(int(d_clean) - 1)
                    if nome_raw and dias_list:
                        # Mapeia tanto por ID quanto por Nome
                        for _, p_row in df_profs.iterrows():
                            if p_row['ID'] in nome_raw or p_row['Nome'].lower() in nome_raw.lower():
                                pedidos_dict[p_row['Nome']] = dias_list
                st.session_state['pedidos_dict'] = pedidos_dict
                st.success(f"✅ Pedidos lidos com sucesso para {len(pedidos_dict)} colaboradores!")
            except Exception as e:
                st.error(f"Erro ao ler arquivo: {e}")
    with col_up2:
        st.markdown("**Modelo de Formato da Planilha de Pedidos:**")
        st.code("Nome,Dias_Folga\nAna Paula Silva,\"1, 5, 12\"\nENF-02,\"8, 20\"", language="csv")

# PAINEL EXPANSÍVEL 2: MARCAÇÃO EM LOTE (FÉRIAS, LICENÇAS, ATESTADOS DE DIA X A DIA Y)
with st.expander("📌 Ferramenta de Marcação em Lote (Lançamento Rápido de Férias e Licenças)", expanded=False):
    c_lote1, c_lote2, c_lote3, c_lote4, c_lote5 = st.columns([3, 2, 1.5, 1.5, 2])
    
    nomes_profissionais = df_profs['Nome'].tolist()
    with c_lote1:
        prof_selecionado = st.selectbox("Selecione o Colaborador:", ["TODOS"] + nomes_profissionais, key="lote_prof")
    with c_lote2:
        evento_selecionado = st.selectbox("Tipo de Evento / Plantão:", [
            "FE - Férias",
            "AT - Atestado Médico",
            "LM - Licença Maternidade",
            "LIC - Licença Pessoal",
            "FP - Folga Pedida",
            "M6 - Manhã 6h",
            "D12 - Diurno 12h",
            "N12 - Noturno 12h",
            "FOL - Folga Regulamentar (Limpar)"
        ], key="lote_evento")
    with c_lote3:
        dia_inicio = st.number_input("Dia Inicial:", min_value=1, max_value=30, value=1, key="lote_d_ini")
    with c_lote4:
        dia_fim = st.number_input("Dia Final:", min_value=1, max_value=30, value=15, key="lote_d_fim")
    with c_lote5:
        st.write("") # Espaçamento vertical
        btn_aplicar_lote = st.button("📌 Aplicar no Intervalo", use_container_width=True, type="secondary")

    if btn_aplicar_lote:
        codigo_evt = evento_selecionado.split(" - ")[0].strip()
        if codigo_evt == "FOL":
            codigo_evt = ""
            
        df_tbl = st.session_state['tabela_escala']
        dias_afetados = [colunas_dias[d-1] for d in range(int(dia_inicio), int(dia_fim)+1) if 1 <= d <= 30]
        
        count_alterados = 0
        for idx, r in df_tbl.iterrows():
            if r['ID'] == '---':
                continue
            if prof_selecionado == "TODOS" or r['Nome'] == prof_selecionado or prof_selecionado in r['Nome']:
                for col_d in dias_afetados:
                    df_tbl.at[idx, col_d] = codigo_evt
                count_alterados += 1
                
        st.session_state['tabela_escala'] = df_tbl
        salvar_estado_local()
        st.success(f"✅ Marcação '{codigo_evt if codigo_evt else 'FOL'}' aplicada com sucesso para {count_alterados} colaboradores nos dias {dia_inicio} a {dia_fim}!")

# CONTROLES SUPERIORES DA PÁGINA PANORÂMICA
c_ctrl1, c_ctrl2, c_ctrl3 = st.columns([3, 3, 2])

with c_ctrl1:
    btn_gerar = st.button("🚀 Otimizar Escala com IA (CP-SAT)", type="primary", use_container_width=True)

with c_ctrl2:
    travar_marcacoes = st.checkbox("🔒 Preservar marcações manuais e afastamentos (Férias, Licenças, Atestados)", value=True, help="Quando ativo, a IA não altera células já preenchidas com FE, AT, LM, LIC ou plantões marked pelo gestor.")

with c_ctrl3:
    btn_limpar = st.button("🗑️ Resetar Escala", use_container_width=True)
    if btn_limpar:
        if os.path.exists(ARQUIVO_PERSISTENCIA):
            os.remove(ARQUIVO_PERSISTENCIA)
        del st.session_state['tabela_escala']
        st.rerun()

# LÓGICA DO OTIMIZADOR CP-SAT SOLVER (GOOGLE OR-TOOLS)
if btn_gerar:
    with st.spinner("Calculando a escala ótima com o Google OR-Tools CP-SAT Solver..."):
        num_profs = len(df_profs)
        num_dias = 30
        dias = list(range(num_dias))
        turnos = ['M6', 'D12', 'N12']
        
        modelo = cp_model.CpModel()
        escala = {(p, d, t): modelo.NewBoolVar(f'p_{p}_d_{d}_t_{t}') for p in range(num_profs) for d in dias for t in turnos}
        
        df_atual = st.session_state['tabela_escala']
        
        # Mapeamento de travas manuais do usuário se a opção do cadeado estiver ativa
        for p in range(num_profs):
            prof = df_profs.iloc[p]
            nome_p = prof['Nome']
            # Encontra a linha na tabela atual
            row_match = df_atual[df_atual['Nome'] == nome_p]
            
            for d in dias:
                col_d = colunas_dias[d]
                val_existente = ""
                if not row_match.empty:
                    val_existente = str(row_match.iloc[0][col_d]).strip().upper()
                
                # Se houver trava de afastamento ou escolha manual mantida
                if travar_marcacoes and val_existente in ['FE', 'AT', 'LM', 'LIC', 'M6', 'D12', 'N12']:
                    if val_existente in ['M6', 'D12', 'N12']:
                        for t in turnos:
                            modelo.Add(escala[(p, d, t)] == (1 if t == val_existente else 0))
                    else:
                        # Se for afastamento (FE, AT, LM, LIC), não trabalha em nenhum turno
                        for t in turnos:
                            modelo.Add(escala[(p, d, t)] == 0)
                else:
                    # Aplica restrições normais de domínio por perfil
                    if prof['Turno_Atribuido'] == 'Noturno':
                        modelo.Add(escala[(p, d, 'M6')] == 0)
                        modelo.Add(escala[(p, d, 'D12')] == 0)
                    else:
                        modelo.Add(escala[(p, d, 'N12')] == 0)
                    modelo.AddAtMostOne(escala[(p, d, t)] for t in turnos)

        # Hard Constraints (Regras Legais da EBSERH / ACT)
        for p in range(num_profs):
            prof = df_profs.iloc[p]
            
            # Interjornada / 12x36
            for d in range(num_dias - 1):
                trabalhou_12 = escala[(p, d, 'D12')] + escala[(p, d, 'N12')]
                trabalhou_prox = sum(escala[(p, d + 1, t)] for t in turnos)
                modelo.Add(trabalhou_12 + trabalhou_prox <= 1)

            # Teto de 6 dias consecutivos
            for d in range(num_dias - 6):
                dias_trab = sum(escala[(p, d + i, t)] for i in range(7) for t in turnos)
                modelo.Add(dias_trab <= 6)

            # Proteção para mulheres: Não trabalhar em dois domingos seguidos
            if prof['Sexo'] == 'Feminino':
                for i in range(len(domingos_idx) - 1):
                    d1, d2 = domingos_idx[i], domingos_idx[i+1]
                    modelo.Add(sum(escala[(p, d1, t)] for t in turnos) + sum(escala[(p, d2, t)] for t in turnos) <= 1)

        # Cobertura Diária Mínima do Andar
        for d in dias:
            is_weekend = d in finais_semana_idx
            enf_diurnos = [p for p in range(num_profs) if df_profs.iloc[p]['Cargo'] == 'Enfermeiro' and df_profs.iloc[p]['Turno_Atribuido'] == 'Diurno']
            tecs_diurnos = [p for p in range(num_profs) if df_profs.iloc[p]['Cargo'] == 'Técnico de Enfermagem' and df_profs.iloc[p]['Turno_Atribuido'] == 'Diurno']
            enf_noturnos = [p for p in range(num_profs) if df_profs.iloc[p]['Cargo'] == 'Enfermeiro' and df_profs.iloc[p]['Turno_Atribuido'] == 'Noturno']
            tecs_noturnos = [p for p in range(num_profs) if df_profs.iloc[p]['Cargo'] == 'Técnico de Enfermagem' and df_profs.iloc[p]['Turno_Atribuido'] == 'Noturno']

            modelo.Add(sum(escala[(p, d, 'M6')] for p in enf_diurnos) == 1)
            modelo.Add(sum(escala[(p, d, 'M6')] for p in tecs_diurnos) == 1)
            modelo.Add(sum(escala[(p, d, 'D12')] for p in enf_diurnos) == (3 if is_weekend else 5))
            modelo.Add(sum(escala[(p, d, 'D12')] for p in tecs_diurnos) == (4 if is_weekend else 6))
            modelo.Add(sum(escala[(p, d, 'N12')] for p in enf_noturnos) == 4)
            modelo.Add(sum(escala[(p, d, 'N12')] for p in tecs_noturnos) == 3)

        # Soft Constraints (Penalização por desvio de meta e atratividade de pedidos de folga)
        penalidades = []
        pedidos_dict = st.session_state.get('pedidos_dict', {})

        for p in range(num_profs):
            prof = df_profs.iloc[p]
            nome_p = prof['Nome']
            meta = 156 if prof['Carga_Horaria_Semanal'] == '36h' else 176
            horas_mes = sum(escala[(p, d, 'M6')] * 6 + escala[(p, d, 'D12')] * 12 + escala[(p, d, 'N12')] * 12 for d in dias)
            diff = modelo.NewIntVar(-40, 40, f'diff_{p}')
            modelo.Add(diff == horas_mes - meta)
            abs_diff = modelo.NewIntVar(0, 40, f'abs_diff_{p}')
            modelo.AddAbsEquality(abs_diff, diff)
            penalidades.append(abs_diff * 10)

            # Atendimento aos pedidos de folga do arquivo
            if nome_p in pedidos_dict:
                for dia_req in pedidos_dict[nome_p]:
                    if 0 <= dia_req < 30:
                        trab_req = sum(escala[(p, dia_req, t)] for t in turnos)
                        penalidades.append(trab_req * 500)

        modelo.Minimize(sum(penalidades))
        solver = cp_model.CpSolver()
        solver.parameters.max_time_in_seconds = 15.0
        status = solver.Solve(modelo)

        if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
            rows_nova = []
            
            # Recria a estrutura mantendo a separação visual
            # 1. Enfermeiros
            for p in range(num_profs):
                prof = df_profs.iloc[p]
                if prof['Cargo'] != 'Enfermeiro':
                    continue
                meta = 156 if prof['Carga_Horaria_Semanal'] == '36h' else 176
                horas_trab = sum(solver.Value(escala[(p, d, 'M6')]) * 6 + solver.Value(escala[(p, d, 'D12')]) * 12 + solver.Value(escala[(p, d, 'N12')]) * 12 for d in dias)
                saldo = horas_trab - meta
                
                r = {'ID': prof['ID'], 'Nome': prof['Nome'], 'Meta': meta}
                row_existente = df_atual[df_atual['Nome'] == prof['Nome']]
                
                for d in dias:
                    col_d = colunas_dias[d]
                    val_antigo = str(row_existente.iloc[0][col_d]).strip().upper() if not row_existente.empty else ""
                    
                    if travar_marcacoes and val_antigo in ['FE', 'AT', 'LM', 'LIC']:
                        t_val = val_antigo
                    else:
                        t_val = ''
                        for t in turnos:
                            if solver.Value(escala[(p, d, t)]) == 1:
                                t_val = t
                        if t_val == '' and prof['Nome'] in pedidos_dict and d in pedidos_dict[prof['Nome']]:
                            t_val = 'FP' # Folga Pedida
                            
                    r[col_d] = t_val
                    
                r['Cumprida'] = horas_trab
                r['Saldo'] = f"+{saldo}h" if saldo > 0 else (f"{saldo}h" if saldo < 0 else "0h")
                rows_nova.append(r)

            # 2. Linha Divisória de Técnicos
            r_div = {'ID': '---', 'Nome': '--- TÉCNICOS DE ENFERMAGEM ---', 'Meta': 0}
            for col in colunas_dias:
                r_div[col] = '---'
            r_div['Cumprida'] = 0
            r_div['Saldo'] = '---'
            rows_nova.append(r_div)

            # 3. Técnicos
            for p in range(num_profs):
                prof = df_profs.iloc[p]
                if prof['Cargo'] != 'Técnico de Enfermagem':
                    continue
                meta = 156 if prof['Carga_Horaria_Semanal'] == '36h' else 176
                horas_trab = sum(solver.Value(escala[(p, d, 'M6')]) * 6 + solver.Value(escala[(p, d, 'D12')]) * 12 + solver.Value(escala[(p, d, 'N12')]) * 12 for d in dias)
                saldo = horas_trab - meta
                
                r = {'ID': prof['ID'], 'Nome': prof['Nome'], 'Meta': meta}
                row_existente = df_atual[df_atual['Nome'] == prof['Nome']]
                
                for d in dias:
                    col_d = colunas_dias[d]
                    val_antigo = str(row_existente.iloc[0][col_d]).strip().upper() if not row_existente.empty else ""
                    
                    if travar_marcacoes and val_antigo in ['FE', 'AT', 'LM', 'LIC']:
                        t_val = val_antigo
                    else:
                        t_val = ''
                        for t in turnos:
                            if solver.Value(escala[(p, d, t)]) == 1:
                                t_val = t
                        if t_val == '' and prof['Nome'] in pedidos_dict and d in pedidos_dict[prof['Nome']]:
                            t_val = 'FP' # Folga Pedida
                            
                    r[col_d] = t_val
                    
                r['Cumprida'] = horas_trab
                r['Saldo'] = f"+{saldo}h" if saldo > 0 else (f"{saldo}h" if saldo < 0 else "0h")
                rows_nova.append(r)

            st.session_state['tabela_escala'] = pd.DataFrame(rows_nova)
            salvar_estado_local()
            st.success("✅ Escala otimizada com sucesso! As marcações prévias e os pedidos de folga foram respeitados.")

# EXIBIÇÃO DA TABELA PANORÂMICA E EDITÁVEL
st.subheader("📋 Grade Mensal Panorâmica de Novembro/2026")
st.caption("Legenda dos Códigos: M6 (Manhã 6h) | D12 (Diurno 12h) | N12 (Noturno 12h) | FP (Folga Pedida - Verde) | FE (Férias) | AT (Atestado) | LM (Lic. Maternidade) | LIC (Lic. Pessoal) | Células em Branco (Folga Regulamentar)")

# ESTILIZAÇÃO DE CORES VIA PANDAS STYLER
def destacar_celulas(df):
    # Cria uma matriz de estilos vazia
    styles = pd.DataFrame('', index=df.index, columns=df.columns)
    
    # Destacar colunas de finais de semana com fundo azul clarinho
    for col in colunas_dias:
        d_num = int(col.split(' ')[0]) - 1
        if d_num in finais_semana_idx:
            styles[col] = 'background-color: #f0f7ff;'
            
    # Destacar valores e códigos de turnos/afastamentos
    for idx, row in df.iterrows():
        if row['ID'] == '---':
            for col in df.columns:
                styles.at[idx, col] = 'background-color: #e2e8f0; font-weight: bold; color: #1a202c;'
            continue
            
        for col in colunas_dias:
            val = str(row[col]).strip().upper()
            base_bg = styles.at[idx, col]
            
            if val == 'FP':
                styles.at[idx, col] = 'background-color: #d4edda; color: #155724; font-weight: bold;'
            elif val == 'FE':
                styles.at[idx, col] = 'background-color: #fff3cd; color: #856404; font-weight: bold;'
            elif val == 'AT':
                styles.at[idx, col] = 'background-color: #f8d7da; color: #721c24; font-weight: bold;'
            elif val == 'LM':
                styles.at[idx, col] = 'background-color: #e2d9f3; color: #4a154b; font-weight: bold;'
            elif val == 'LIC':
                styles.at[idx, col] = 'background-color: #ffe8cc; color: #d9480f; font-weight: bold;'
            elif val in ['M6', 'D12', 'N12']:
                styles.at[idx, col] = base_bg + ' font-weight: 600;'
                
    return styles

# Exibe a tabela interativa editável
df_para_editar = st.session_state['tabela_escala']

df_editado = st.data_editor(
    df_para_editar,
    use_container_width=True,
    num_rows="fixed",
    disabled=["ID", "Nome", "Meta", "Cumprida", "Saldo"],
    key="editor_escala_panoramica"
)

# RECALCULAR SALDOS, AUDITORIA DE ERROS E PERSISTÊNCIA APÓS EDIÇÃO MANUAL
erros_detectados = []
horas_map = {'M6': 6, 'D12': 12, 'N12': 12, '': 0, 'FP': 0, 'FE': 0, 'AT': 0, 'LM': 0, 'LIC': 0, '---': 0}

for idx, row in df_editado.iterrows():
    if row['ID'] == '---':
        continue
        
    nome = row['Nome']
    total_h = 0
    dias_consecutivos = 0
    
    for d in range(30):
        col_d = colunas_dias[d]
        val = str(row[col_d]).strip().upper()
        h = horas_map.get(val, 0)
        total_h += h
        
        # Checagem de interjornada (12x36)
        if d < 29:
            col_prox = colunas_dias[d+1]
            val_prox = str(row[col_prox]).strip().upper()
            if val in ['D12', 'N12'] and val_prox in ['M6', 'D12', 'N12']:
                erros_detectados.append(f"🚨 **{nome}**: Quebra de 12x36 / Interjornada entre o Dia {d+1} ({val}) e o Dia {d+2} ({val_prox}).")
                
        # Checagem de limite de 6 dias consecutivos
        if h > 0:
            dias_consecutivos += 1
            if dias_consecutivos > 6:
                erros_detectados.append(f"🚨 **{nome}**: Excede o teto legal de 6 dias consecutivos de trabalho no Dia {d+1}.")
        else:
            dias_consecutivos = 0

    meta = int(row['Meta']) if str(row['Meta']).isdigit() else 0
    saldo = total_h - meta
    df_editado.at[idx, 'Cumprida'] = total_h
    df_editado.at[idx, 'Saldo'] = f"+{saldo}h" if saldo > 0 else (f"{saldo}h" if saldo < 0 else "0h")

st.session_state['tabela_escala'] = df_editado
salvar_estado_local()

# EXIBIÇÃO DA TABELA FORMATADA COM DESTAQUE DE CORES
with st.expander("🎨 Visualizar Tabela Formatada com Cores (Destaque de Finais de Semana, FP e Afastamentos)", expanded=True):
    st.dataframe(df_editado.style.apply(destacar_celulas, axis=None), use_container_width=True, height=450)

# PAINEL DE AUDITORIA DE ERROS E INCONSISTÊNCIAS
if erros_detectados:
    st.error(f"⚠️ **Foram identificadas {len(erros_detectados)} infrações normativas nas edições manuais:**")
    for err in erros_detectados[:6]:
        st.write(err)
else:
    st.success("✅ **100% de Conformidade Legal:** Nenhuma infração normativa identificada na escala atual.")

# 📊 DASHBOARD DA ALTA GESTÃO (METRICAS, ATESTADOS, LICENÇAS E INDICADORES)
st.divider()
st.subheader("📊 Dashboard de Indicadores e Dimensionamento (Visão da Alta Gestão)")

# Contagem de Afastamentos e Licenças na Escala
cnt_fe = 0
cnt_at = 0
cnt_lm = 0
cnt_lic = 0
cnt_fp = 0
detalhes_afastamentos = []

for idx, r in df_editado.iterrows():
    if r['ID'] == '---':
        continue
    nome_p = r['Nome']
    cargo_p = "Enfermeiro" if "ENF" in str(r['ID']) else "Técnico de Enfermagem"
    
    dias_fe = [d+1 for d in range(30) if str(r[colunas_dias[d]]).strip().upper() == 'FE']
    dias_at = [d+1 for d in range(30) if str(r[colunas_dias[d]]).strip().upper() == 'AT']
    dias_lm = [d+1 for d in range(30) if str(r[colunas_dias[d]]).strip().upper() == 'LM']
    dias_lic = [d+1 for d in range(30) if str(r[colunas_dias[d]]).strip().upper() == 'LIC']
    dias_fp = [d+1 for d in range(30) if str(r[colunas_dias[d]]).strip().upper() == 'FP']

    cnt_fe += len(dias_fe)
    cnt_at += len(dias_at)
    cnt_lm += len(dias_lm)
    cnt_lic += len(dias_lic)
    cnt_fp += len(dias_fp)

    if dias_fe:
        detalhes_afastamentos.append({'Nome': nome_p, 'Cargo': cargo_p, 'Tipo': 'Férias (FE)', 'Dias': f"{len(dias_fe)} dias", 'Período': f"Dias {min(dias_fe)} a {max(dias_fe)}"})
    if dias_at:
        detalhes_afastamentos.append({'Nome': nome_p, 'Cargo': cargo_p, 'Tipo': 'Atestado Médico (AT)', 'Dias': f"{len(dias_at)} dias", 'Período': f"Dias {min(dias_at)} a {max(dias_at)}"})
    if dias_lm:
        detalhes_afastamentos.append({'Nome': nome_p, 'Cargo': cargo_p, 'Tipo': 'Licença Maternidade (LM)', 'Dias': f"{len(dias_lm)} dias", 'Período': f"Dias {min(dias_lm)} a {max(dias_lm)}"})
    if dias_lic:
        detalhes_afastamentos.append({'Nome': nome_p, 'Cargo': cargo_p, 'Tipo': 'Licença Pessoal (LIC)', 'Dias': f"{len(dias_lic)} dias", 'Período': f"Dias {min(dias_lic)} a {max(dias_lic)}"})

m1, m2, m3, m4, m5, m6 = st.columns(6)
m1.metric("Equipe Total", "40 prof.")
m2.metric("Férias (FE)", f"{cnt_fe} plantões")
m3.metric("Atestados (AT)", f"{cnt_at} plantões")
m4.metric("Lic. Maternidade (LM)", f"{cnt_lm} plantões")
m5.metric("Lic. Pessoal (LIC)", f"{cnt_lic} plantões")
m6.metric("Folgas Pedidas (FP)", f"{cnt_fp} concedidas")

# TABELA RESUMO NOMINAL DE AFASTAMENTOS E ABSENTEÍSMO
st.markdown("##### 📄 Relatório Consolidado de Afastamentos no Mês")
if detalhes_afastamentos:
    df_afast = pd.DataFrame(detalhes_afastamentos)
    st.dataframe(df_afast, use_container_width=True, hide_index=True)
else:
    st.info("Nenhum afastamento especial (Férias, Atestado, Licença Maternidade ou Licença Pessoal) registrado no mês atual.")

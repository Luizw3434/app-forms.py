import sqlite3
import tkinter as tk
from tkinter import messagebox
from tkinter import ttk


# Configuração e Inicialização do Banco de Dados (SQLite)
def inicializar_banco():
    conn = sqlite3.connect("clientes.db")
    cursor = conn.cursor()
    # Cria a tabela conforme especificado nas questões da prova
    cursor.execute(
        """
        CREATE TABLE IF NOT EXISTS clientes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nome TEXT NOT NULL,
            email TEXT NOT NULL,
            telefone TEXT NOT NULL
        )
    """
    )
    conn.commit()
    conn.close()


# Função para salvar os dados do cliente no banco de dados
def salvar_cliente():
    nome = entry_nome.get().strip()
    email = entry_email.get().strip()
    telefone = entry_telefone.get().strip()

    # Validação: Garanta que todos os campos sejam preenchidos
    if not nome or not email or not telefone:
        messagebox.showwarning(
            "Atenção", "Todos os campos devem ser preenchidos!"
        )
        return

    try:
        conn = sqlite3.connect("clientes.db")
        cursor = conn.cursor()
        cursor.execute(
            "INSERT INTO clientes (nome, email, telefone) VALUES (?, ?, ?)",
            (nome, email, telefone),
        )
        conn.commit()
        conn.close()

        messagebox.showinfo("Sucesso", "Cliente salvo com sucesso!")
        limpar_formulario()  # Limpa o formulário após salvar

    except Exception as e:
        messagebox.showerror(
            "Erro", f"Ocorreu um erro ao salvar no banco de dados: {e}"
        )


# Função para limpar o formulário
def limpar_formulario():
    entry_nome.delete(0, tk.END)
    entry_email.delete(0, tk.END)
    entry_telefone.delete(0, tk.END)


# Função para visualizar os dados dos clientes em uma nova janela
def visualizar_clientes():
    janela_visualizacao = tk.Toplevel()
    janela_visualizacao.title("Clientes Cadastrados")
    janela_visualizacao.geometry("500x300")

    # Estrutura de tabela Treeview com 4 colunas para exibição
    tree = ttk.Treeview(
        janela_visualizacao,
        columns=("ID", "Nome", "Email", "Telefone"),
        show="headings",
    )
    tree.heading("ID", text="ID")
    tree.heading("Nome", text="Nome")
    tree.heading("Email", text="Email")
    tree.heading("Telefone", text="Telefone")

    # Ajuste de largura das colunas
    tree.column("ID", width=50, anchor="center")
    tree.column("Nome", width=150)
    tree.column("Email", width=150)
    tree.column("Telefone", width=120)

    tree.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

    try:
        conn = sqlite3.connect("clientes.db")
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM clientes")
        for row in cursor.fetchall():
            tree.insert("", tk.END, values=row)
        conn.close()
    except Exception as e:
        messagebox.showerror("Erro", f"Erro ao carregar dados: {e}")


# Inicializa a estrutura do banco de dados antes de abrir a janela
inicializar_banco()

# Configuração da Interface Gráfica Principal (Tkinter)
janela = tk.Tk()
janela.title("Cadastro de Clientes")
janela.geometry("400x280")
janela.resizable(False, False)

# Configuração de layout estruturado
padding_options = {"padx": 15, "pady": 8}

# Rótulos (Labels) e Campos de Entrada (Entries)
tk.Label(janela, text="Nome:").grid(row=0, column=0, sticky="w", **padding_options)
entry_nome = tk.Entry(janela, width=32)
entry_nome.grid(row=0, column=1, **padding_options)

tk.Label(janela, text="E-mail:").grid(row=1, column=0, sticky="w", **padding_options)
entry_email = tk.Entry(janela, width=32)
entry_email.grid(row=1, column=1, **padding_options)

tk.Label(janela, text="Telefone:").grid(
    row=2, column=0, sticky="w", **padding_options
)
entry_telefone = tk.Entry(janela, width=32)
entry_telefone.grid(row=2, column=1, **padding_options)

# Contêiner para organizar os botões alinhados na parte inferior
frame_botoes = tk.Frame(janela)
frame_botoes.grid(row=3, column=0, columnspan=2, pady=20)

# Botões do formulário (Salvar, Limpar e o botão Visualizar exigido na questão 3)
btn_salvar = tk.Button(
    frame_botoes, text="Salvar", command=salvar_cliente, bg="#4CAF50", fg="white"
)
btn_salvar.pack(side="left", padx=5)

btn_limpar = tk.Button(
    frame_botoes, text="Limpar", command=limpar_formulario, bg="#f44336", fg="white"
)
btn_limpar.pack(side="left", padx=5)

btn_visualizar = tk.Button(
    frame_botoes,
    text="Visualizar Clientes",
    command=visualizar_clientes,
    bg="#2196F3",
    fg="white",
)
btn_visualizar.pack(side="left", padx=5)

# Inicia o loop da aplicação gráfica
janela.mainloop()

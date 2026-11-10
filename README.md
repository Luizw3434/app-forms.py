import sqlite3
import tkinter as tk
from tkinter import messagebox, ttk


# --- CONFIGURAÇÃO DO BANCO DE DADOS ---
def init_db():
    conn = sqlite3.connect("cadastro_clientes.db")
    cursor = conn.cursor()
    cursor.execute(
        """
        CREATE TABLE IF NOT EXISTS clientes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nome TEXT NOT NULL,
            email TEXT NOT NULL,
            telefone TEXT
        )
    """
    )
    conn.commit()
    conn.close()


# --- FUNÇÕES DO SISTEMA ---
def cadastrar_cliente():
    nome = entry_nome.get()
    email = entry_email.get()
    telefone = entry_telefone.get()

    if not nome or not email:
        messagebox.showwarning(
            "Aviso", "Os campos Nome e E-mail são obrigatórios!"
        )
        return

    conn = sqlite3.connect("cadastro_clientes.db")
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO clientes (nome, email, telefone) VALUES (?, ?, ?)",
        (nome, email, telefone),
    )
    conn.commit()
    conn.close()

    messagebox.showinfo("Sucesso", "Cliente cadastrado com sucesso!")

    # Limpa os campos após o cadastro
    entry_nome.delete(0, tk.END)
    entry_email.delete(0, tk.END)
    entry_telefone.delete(0, tk.END)


# REQUISITO A INCLUIR: Funcionalidade "Visualizar Clientes"
def abrir_janela_visualizacao():
    # Cria uma nova janela (Toplevel)
    janela_visualizar = tk.Toplevel(root)
    janela_visualizar.title("Clientes Cadastrados")
    janela_visualizar.geometry("600x400")

    # Título interno
    lbl_titulo = tk.Label(
        janela_visualizar,
        text="Histórico de Clientes",
        font=("Arial", 14, "bold"),
    )
    lbl_titulo.pack(pady=10)

    # Criação da tabela (Treeview) para mostrar os dados
    colunas = ("ID", "Nome", "E-mail", "Telefone")
    tabela = ttk.Treeview(
        janela_visualizar, columns=colunas, show="headings", height=15
    )

    # Define os cabeçalhos das colunas
    tabela.heading("ID", text="ID")
    tabela.heading("Nome", text="Nome")
    tabela.heading("E-mail", text="E-mail")
    tabela.heading("Telefone", text="Telefone")

    # Define a largura das colunas
    tabela.column("ID", width=50, anchor="center")
    tabela.column("Nome", width=180)
    tabela.column("E-mail", width=200)
    tabela.column("Telefone", width=120)

    # Busca os dados no banco de dados para popular a tabela
    conn = sqlite3.connect("cadastro_clientes.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM clientes")
    rows = cursor.fetchall()

    for row in rows:
        tabela.insert("", tk.END, values=row)

    conn.close()

    # Adiciona uma barra de rolagem
    scrollbar = ttk.Scrollbar(
        janela_visualizar, orient=tk.VERTICAL, command=tabela.yview
    )
    tabela.configure(yscrollcommand=scrollbar.set)

    # Empacota a tabela e a barra de rolagem na janela
    tabela.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(10, 0), pady=10)
    scrollbar.pack(side=tk.RIGHT, fill=tk.Y, padx=(0, 10), pady=10)


# --- INTERFACE GRÁFICA PRINCIPAL ---
init_db()  # Garante que o banco de dados e a tabela existam

root = tk.Tk()
root.title("Formulário de Cadastro de Clientes")
root.geometry("400x300")

# Elementos visuais da janela principal
lbl_nome = tk.Label(root, text="Nome:", font=("Arial", 10))
lbl_nome.pack(pady=(15, 2))
entry_nome = tk.Entry(root, width=40)
entry_nome.pack()

lbl_email = tk.Label(root, text="E-mail:", font=("Arial", 10))
lbl_email.pack(pady=(10, 2))
entry_email = tk.Entry(root, width=40)
entry_email.pack()

lbl_telefone = tk.Label(root, text="Telefone:", font=("Arial", 10))
lbl_telefone.pack(pady=(10, 2))
entry_telefone = tk.Entry(root, width=40)
entry_telefone.pack()

# Botão para salvar o cadastro
btn_cadastrar = tk.Button(
    root,
    text="Cadastrar Cliente",
    command=cadastrar_cliente,
    bg="#4CAF50",
    fg="white",
    font=("Arial", 10, "bold"),
)
btn_cadastrar.pack(pady=20)

# Botão acrescentado para atender ao requisito do exercício
btn_visualizar = tk.Button(
    root,
    text="Visualizar Clientes",
    command=abrir_janela_visualizacao,
    bg="#2196F3",
    fg="white",
    font=("Arial", 10, "bold"),
)
btn_visualizar.pack()

root.mainloop()

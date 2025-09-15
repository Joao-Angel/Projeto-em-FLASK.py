from flask import Flask, render_template, request, redirect, url_for
import csv
import os
from datetime import datetime  #datas

app = Flask(__name__)

# Lista global para armazenar despesas temporariamente
despesas = []

# Nome do arquivo CSV
CSV_FILE = "despesas.csv"

# Função para carregar despesas do CSV ao iniciar
def carregar_csv():
    if os.path.isfile(CSV_FILE):
        with open(CSV_FILE, mode="r", newline="", encoding="utf-8") as f:
            reader = csv.DictReader(f)
            for row in reader:
                # Converte valores para o tipo correto
                despesa = {
                    "id": int(row["id"]),
                    "descricao": row["descricao"],
                    "valor": float(row["valor"]),
                    "categoria": row["categoria"],
                    "data": row.get("data", "")  # lê a data se existir
                }
                despesas.append(despesa)

# Função para salvar uma nova despesa (append)
def salvar_no_csv(despesa):
    arquivo_existe = os.path.isfile(CSV_FILE)
    with open(CSV_FILE, mode="a", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["id", "descricao", "valor", "categoria", "data"])
        if not arquivo_existe:
            writer.writeheader()  # escreve cabeçalho se não existir
        writer.writerow(despesa)

# Função para reescrever o CSV inteiro (usada em editar/remover)
def reescrever_csv():
    with open(CSV_FILE, mode="w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["id", "descricao", "valor", "categoria", "data"])
        writer.writeheader()
        for despesa in despesas:
            writer.writerow(despesa)

@app.route("/")
def home():
    # Calcula o total de todas as despesas
    total = sum(d['valor'] for d in despesas)

    # Calcula total por categoria
    categorias = {}
    for d in despesas:
        categoria = d.get('categoria', 'Sem categoria')
        categorias[categoria] = categorias.get(categoria, 0) + d['valor']

    # Passa tudo para o template
    return render_template("index.html", despesas=despesas, total=total, categorias=categorias)

@app.route('/adicionar', methods=['GET', 'POST'])
def adicionar():
    if request.method == 'POST':
        # Pega dados do formulário
        descricao = request.form.get("descricao")#novo campo para descrição
        valor = request.form.get("valor")#novo campo para valor
        categoria = request.form.get("categoria")  # Novo campo para categoria
        data = request.form.get("data")  # Novo campo para data

        if not data:
            # Se não preencher, usa data atual
            data = datetime.now().strftime("%Y-%m-%d")

        # Cria um dicionário representando a despesa
        despesa = {
            "id": len(despesas) + 1,
            "descricao": descricao,
            "valor": float(valor),
            "categoria": categoria,  
            "data": data
        }

        # Adiciona na lista global
        despesas.append(despesa)

        # Salva a despesa no CSV
        salvar_no_csv(despesa)

        # Redireciona para a página inicial
        return redirect(url_for("home"))

    # Se for GET, exibe o formulário
    return render_template("adicionar.html")

@app.route('/despesa/<int:id>')
def detalhe(id):
    # Busca a despesa pelo id
    for despesa in despesas:
        if despesa["id"] == id:
            return f"Despesa: {despesa['descricao']} - R${despesa['valor']:.2f} - Categoria: {despesa.get('categoria', 'Sem categoria')} - Data: {despesa.get('data', '')}"
    return "Despesa não encontrada", 404

@app.route('/remover/<int:id>', methods=['POST'])
def remover(id):
    global despesas
    # Filtra a lista, removendo a despesa com o id fornecido
    despesas = [d for d in despesas if d["id"] != id]
    # Reescreve o CSV atualizado
    reescrever_csv()
    # Redireciona para a página inicial
    return redirect(url_for("home"))

@app.route('/editar/<int:id>', methods=['GET', 'POST'])
def editar(id):
    # Busca a despesa pelo id
    despesa = next((d for d in despesas if d["id"] == id), None)
    if not despesa:
        return "Despesa não encontrada", 404

    if request.method == 'POST':
        # Atualiza os valores com os dados do formulário
        despesa['descricao'] = request.form.get("descricao")
        despesa['valor'] = float(request.form.get("valor"))
        despesa['categoria'] = request.form.get("categoria")  # Atualiza categoria
        data = request.form.get("data")
        if not data:
            data = datetime.now().strftime("%Y-%m-%d")
        despesa['data'] = data

        # Reescreve o CSV atualizado
        reescrever_csv()

        return redirect(url_for("home"))

    # Se for GET, mostra o formulário de edição preenchido
    return render_template("editar.html", despesa=despesa)

if __name__ == "__main__":
    # Carrega despesas do CSV ao iniciar
    carregar_csv()
    app.run(debug=True)

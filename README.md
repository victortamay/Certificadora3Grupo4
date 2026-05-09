# Roteiro Grupo 4

## Parte 1: Compilação e Execução do Sistema

### 1.1 Ferramentas de Desenvolvimento

- **Editor de Código:** Visual Studio Code (v1.85+) | [code.visualstudio.com](https://code.visualstudio.com)
    
- **Execução/Ambiente:** Navegador Moderno (Chrome 120+, Firefox 120+) com extensão **Live Server** para emulação de servidor local.
    
- **Compilação:** O projeto utiliza **JavaScript Vanilla (ES6+)** e não requer um passo de compilação pesado (Build Step), sendo interpretado diretamente pelo motor V8 do navegador.
    

### 1.2 Base de Dados e Hospedagem

- **Ferramenta:** **Supabase** (Backend as a Service - BaaS).
    
- **Versão:** API v2.
    
- **Link:** [supabase.com](https://supabase.com).
    

### 1.3 Bibliotecas e Ferramentas Complementares

- **Tailwind CSS (v3.4):** Framework de estilização via CDN para design responsivo.
    
- **Lucide Icons:** Biblioteca de ícones vetoriais.
    
- **Supabase JS SDK (v2.42):** Biblioteca cliente para comunicação com o banco de dados e autenticação.
    

### 1.4 Roteiro para Criação da Base de Dados (Caso queira criar um novo )

1. **Criar Projeto:** No console do Supabase, criar um novo projeto.
    
2. **Schema de Tabelas:** Executar o SQL Editor para criar as tabelas:
    
    - `activities`: (id, title, description, type, start_date, total_slots, available_slots, location).
        
    - `registrations`: (id, user_id, activity_id, status).
        
    - `user_roles`: (user_id, role).
        
3. **Configuração de RPC:** Criar as funções de banco de dados (`register_for_activity` e `cancel_registration`) para garantir a atomicidade na contagem de vagas.
    
4. **Habilitar Auth:** Configurar o provedor de Email/Senha no menu _Authentication_.
    

### 1.5 Roteiro para Execução da Aplicação

1. **Repositório:** Utilizar o link para acessar o projeto: https://github.com/victortamay/Certificadora3Grupo4

2. **Arquivos:** Salvar os arquivos `index.html`, `script.js` e `style.css` na mesma pasta de seu computador.
    
3. **Configuração:** Caso tenha criado um novo banco e queira usa-lo. Inserir a `SUPABASE_URL` e a `SUPABASE_KEY` no início do arquivo `script.js`.
    
4. **Execução:** Abrir o `index.html` utilizando a extensão **Live Server** do VS Code para garantir que as rotas e o protocolo `https` da API funcionem corretamente.
    

---

## Parte 2: Teste e Apresentação do Sistema

### 2.1 Identificação e Objetivo

- **Responsáveis pelo projeto:** Bruno Navarro, Victor Ehiti, Vitor Melin, Vitor Menck.
    
- **Objetivo do Sistema:** Promover oficinas, minicursos e palestras para despertar o interesse de meninas e mulheres pela área de computação e tecnologia.
    

### 2.2 Funcionalidades Desenvolvidas

- **Navegação SPA:** Troca de páginas sem recarregamento do navegador via sistema de hashes.
    
- **Autenticação Completa:** Cadastro de novas usuárias e login de integrantes.
    
- **Gestão de Atividades:** Listagem dinâmica com cálculo de status em tempo real (aberta, lotada ou encerrada).
    
- **Inscrições:** Sistema de reserva de vagas com validação de disponibilidade.
    
### 2.3 Roteiro de Testes

1. **Acesso Inicial:** Visualizar a Home e a página de Atividades como visitante (sem login).
    
2. **Fluxo de Cadastro:** Criar uma nova conta na página de Cadastro.
    
3. **Fluxo de Inscrição:** * Fazer login com a conta criada.
    
    - Escolher uma atividade com status "Aberta".
        
    - Clicar em "Inscrever-se" e verificar o redirecionamento para "Minhas Inscrições".
        
4. **Fluxo de Cancelamento:** Cancelar a inscrição realizada e verificar se a vaga retorna ao sistema.
    

### 2.4 Contas de Acesso Padrão

- **Usuário Comum:** vitorsabre@gmail.com    Senha: abc132

Vamos lá! Para fazer o deploy do seu site React e da API Node.js no Portainer, siga os passos abaixo. O processo envolve a criação de imagens Docker para cada projeto, configuração de um arquivo `docker-compose.yml` e implantação via Portainer. 

---

### **Passo 1: Preparar os projetos para Docker**
#### **React (Frontend)**
1. **Gere a build de produção**:
   - Execute `npm run build` na raiz do projeto React. Isso criará uma pasta `build` com os arquivos estáticos otimizados .
   - **Importante**: Remova `<React.StrictMode>` do `index.js` antes da build para evitar comportamentos inesperados em produção .

2. **Crie um Dockerfile para o React** (exemplo para produção):
   ```dockerfile
   # Estágio de construção
   FROM node:18-alpine AS build
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build

   # Estágio de produção (usando Nginx)
   FROM nginx:stable-alpine
   COPY --from=build /app/build /usr/share/nginx/html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```
   Esse Dockerfile usa um *multi-stage build* para otimizar o tamanho da imagem final .

#### **Node.js (Backend)**
1. **Configure a API para servir arquivos estáticos** (caso queira integrar o frontend):
   ```javascript
   const express = require('express');
   const app = express();
   const path = require('path');

   // Servir arquivos estáticos do React
   app.use(express.static(path.join(__dirname, 'build')));
   app.get('*', (req, res) => {
     res.sendFile(path.join(__dirname, 'build', 'index.html'));
   });

   // Rotas da API
   app.get('/api/data', (req, res) => {
     res.json({ message: 'Dados da API' });
   });

   const PORT = process.env.PORT || 3000;
   app.listen(PORT, () => console.log(`Servidor rodando na porta ${PORT}`));
   ```
   Isso permite que o Node.js sirva tanto a API quanto o frontend React .

2. **Crie um Dockerfile para o Node.js**:
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["node", "server.js"]
   ```

---

### **Passo 2: Configurar o Docker Compose**
Crie um arquivo `docker-compose.yml` para orquestrar os serviços:
```yaml
version: '3.9'
services:
  frontend:
    build: ./caminho/para/pasta-react
    ports:
      - "80:80"  # Nginx (React)
    depends_on:
      - backend

  backend:
    build: ./caminho/para/pasta-node
    ports:
      - "3000:3000"  # API Node.js
    environment:
      - NODE_ENV=production
      - PORT=3000
```
- **Explicação**:
  - `frontend` e `backend` são serviços separados.
  - Use `depends_on` para garantir que o backend inicie antes do frontend .

---

### **Passo 3: Implantar no Portainer**
1. **Suba as imagens para um registro (ex: Docker Hub)**:
   - Construa e marque as imagens:
     ```bash
     docker build -t seuusuario/react-app ./pasta-react
     docker build -t seuusuario/node-api ./pasta-node
     docker push seuusuario/react-app
     docker push seuusuario/node-api
     ```

2. **No Portainer**:
   - Acesse a interface do Portainer (geralmente em `http://seu-servidor:9000`).
   - Vá para **Stacks > Add Stack**.
   - Cole o conteúdo do `docker-compose.yml` no editor.
   - Clique em **Deploy** para iniciar os containers .

---

### **Passo 4: Ajustes Finais**
- **Variáveis de ambiente**: Configure variáveis como `REACT_APP_API_URL=http://backend:3000` no frontend para apontar para a API .
- **Persistência de dados**: Se a API usa banco de dados, adicione um volume no `docker-compose.yml` para persistir dados .
- **Segurança**: Use `.dockerignore` para excluir `node_modules` e arquivos sensíveis (ex: `.env`) .

---

### **Solução para Erros Comuns**
- **"Cannot GET /"**: Configure rotas no Node.js para redirecionar todas as requisições ao React (exemplo no Passo 1) .
- **Portas conflitantes**: Altere as portas no `docker-compose.yml` se houver conflitos (ex: usar `3001:3000` para o backend) .

---

### **Referências Úteis**
- Webpage 4: Exemplo detalhado de Docker Compose para React + Node.js .
- Webpage 6: Guia de Dockerização do React com Nginx .
- Webpage 10: Tutorial de deploy no Portainer .

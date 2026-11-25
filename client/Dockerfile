# Use Node.js
FROM node:20-alpine

# Working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Build the React application
RUN npm run build

# Expose Vite preview port
EXPOSE 4173

# Start preview server
CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0"]
# Instalación de Docker en Ubuntu (WSL2)

## 1. Actualizar paquetes
```bash
sudo apt update && sudo apt upgrade -y
```

<img width="840" height="178" alt="image" src="https://github.com/user-attachments/assets/c30101a6-3ff4-4dce-8b2c-179555078824" />

## 2. Instalar dependencias
```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

<img width="1093" height="461" alt="image" src="https://github.com/user-attachments/assets/86b3e8ad-26f7-4469-ba40-27be7709547f" />

## 3. Añadir clave GPG de Docker
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

<img width="1095" height="72" alt="image" src="https://github.com/user-attachments/assets/1d8a77b2-eb68-4ac5-a5ec-9bb80405d52f" />

## 4. Añadir repositorio de Docker
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

<img width="1102" height="77" alt="image" src="https://github.com/user-attachments/assets/eda6778c-6089-4bb2-b14f-29797e6de5fe" />

## 5. Instalar Docker
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

<img width="1021" height="373" alt="image" src="https://github.com/user-attachments/assets/58025b55-1736-4d61-aa1e-bd0e9d2aa8ba" />

## 6. Iniciar Docker
```bash
sudo systemctl enable --now docker
```

<img width="979" height="79" alt="image" src="https://github.com/user-attachments/assets/322fd815-61ec-4e4f-adfc-182eb4f44391" />

## 7. Permitir usar Docker sin sudo
```bash
sudo usermod -aG docker $USER
```

<img width="653" height="49" alt="image" src="https://github.com/user-attachments/assets/79310a9f-97bd-4257-9041-3af3a55d9ff6" />

## 8. Cerrar sesión y volver a entrar
```bash
exit
```

Volvemos a entrar con:
```bash
wsl
```

<img width="427" height="78" alt="image" src="https://github.com/user-attachments/assets/130bc441-0940-4519-b6b4-34a4b5c04a56" />

## 9. Verificar que funciona
```bash
docker run hello-world
```

<img width="705" height="204" alt="image" src="https://github.com/user-attachments/assets/1c45c401-aea2-4e35-9754-b7373fb80d7d" />





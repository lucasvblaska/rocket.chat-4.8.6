ETAPA 1 — Criar a CA interna (uma vez só)

mkdir -p ~/ca && cd ~/ca

Chave da CA

openssl genrsa -out ca.key 4096

Certificado da CA (10 anos)

openssl req -x509 -new -nodes -key ca.key \
-sha256 -days 3650 -out ca.crt

Quando perguntar, use algo assim:

Country Name: BR
State: SC
Locality: Joinville
Organization Name: GTF
Organizational Unit: TI
Common Name: GTF-CA-INTERNA

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ETAPA 2 — Gerar certificado do Rocket.Chat
Chave privada do servidor

openssl genrsa -out chat.gtf.ind.br.key 2048

Criar o CSR (pedido de certificado)

openssl req -new \
-key chat.gtf.ind.br.key \
-out chat.gtf.ind.br.csr

Use:

Common Name: chat.gtf.ind.br


ETAPA 3 — Criar SAN (OBRIGATÓRIO)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Crie o arquivo san.ext:

cat <<EOF > san.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = chat.gtf.ind.br
IP.1 = SEU_IP_DOCKER
EOF

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ETAPA 4 — Assinar o certificado com a CA

openssl x509 -req \
-in chat.gtf.ind.br.csr \
-CA ca.crt \
-CAkey ca.key \
-CAcreateserial \
-out chat.gtf.ind.br.crt \
-days 825 \
-sha256 \
-extfile san.ext

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ETAPA 5 — Organizar os arquivos

mkdir -p certs
cp chat.gtf.ind.br.crt certs/
cp chat.gtf.ind.br.key certs/
cp ca.crt certs/

Permissões:

chmod 600 certs/chat.gtf.ind.br.key
chmod 644 certs/chat.gtf.ind.br.crt

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ETAPA 6 — Confiar na CA nos clientes (obrigatório)
Windows (manual)
certmgr.msc

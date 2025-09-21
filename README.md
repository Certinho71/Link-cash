# Link-cash
Ganhar com links
const crypto = require('crypto');
const fetch = require('node-fetch');

module.exports = async (req, res) => {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Método não permitido' });
  }

  const { pixKey, valor } = req.body;

  const apikey = process.env.API_KEY;
  const chaveSecreta = process.env.CHAVE_SECRETA;
  const clientId = process.env.CLIENT_ID;
  const tokenAcesso = process.env.TOKEN_ACESSO;
  const payerName = process.env.PAYER_NAME;
  const payerDocument = process.env.PAYER_DOCUMENT;

  const urlPath = `/v1/client/${clientId}/apm/pix/transfer-by-pix-key`;
  const urlBase = 'https://api.pagbrasil.com';
  const timestamp = new Date().toISOString();

  const bodyObject = {
    pixKey,
    value: valor,
    payerName,
    payerDocument,
    description: 'Retirada de ganhos via Pix'
  };

  const requestBody = JSON.stringify(bodyObject);

  function gerarAssinaturaHMAC(apikey, timestamp, requestBody, url, chaveSecreta) {
    const mensagem = apikey + timestamp + requestBody + url;
    const hmac = crypto.createHmac('sha256', chaveSecreta);
    hmac.update(mensagem);
    return hmac.digest('hex');
  }

  const assinatura = gerarAssinaturaHMAC(
    apikey,
    timestamp,
    requestBody,
    urlPath,
    chaveSecreta
  );

  const headers = {
    Authorization: `Bearer ${tokenAcesso}`,
    'Content-Type': 'application/json',
    'x-timestamp': timestamp,
    'x-request-id': crypto.randomUUID(),
    'x-hmac-signature': assinatura
  };

  try {
    const response = await fetch(urlBase + urlPath, {
      method: 'POST',
      headers,
      body: requestBody
    });

    if (!response.ok) {
      const erroData = await response.json();
      return res.status(400).json({ error: erroData.message || 'Erro ao transferir via Pix' });
    }

    const responseData = await response.json();
    return res.status(200).json(responseData);

  } catch (error) {
    console.error('Erro ao transferir via Pix:', error);
    return res.status(500).json({ error: 'Erro interno do servidor' });
  }
};

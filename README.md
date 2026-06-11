# Lista4

# Descrição da Abordagem Adotada

Para identificar de forma não supervisionada os acessos fora do padrão, os dados de log passaram por uma análise estatística e foram submetidos ao algoritmo Isolation Forest com uma taxa de contaminação de 15%.

# Quais padrões de acesso foram considerados normais pelo modelo?

O modelo considerou como padrão "Acesso Normal" a totalidade das interações que ocorreram em horário comercial tradicional (das 9h às 16h). Esse grupo padrão possui características altamente estáveis e previsíveis:Acessos originados de IPs habituais (ip_diferente_habitual = 0).

Apenas 1 única tentativa de login para obter sucesso.

Duração de sessão controlada (variando de 24 a 60 minutos).

Consumo moderado de recursos do sistema (entre 8 e 21 páginas acessadas por sessão).

# Quais características aparecem com maior frequência nos acessos classificados como anômalos?

O Isolation Forest filtrou exatamente 12 anomalias a partir das restrições do modelo. Todos os 12 acessos anômalos compartilham dois gatilhos críticos iniciais: ocorrem estritamente tarde da noite ou de madrugada (das 22h às 3h) e todos partem de um IP diferente do habitual (ip_diferente_habitual = 1). No entanto, ao cruzar o restante das variáveis, descobrimos que o modelo unificou duas assinaturas de anomalias completamente distintas:

Perfil de Volume Atípico (Exfiltração de Dados / Scraping): Sessões de curtíssimo erro de login (apenas 1 tentativa), mas com altíssima duração (120 a 300 minutos) e um volume colossal de páginas acessadas (60 a 120 páginas).

Perfil de Autenticação Suspeita (Ataque de Força Bruta): Sessões com duração insignificante (2 a 6 minutos) e pouquíssimas páginas navegadas (1 a 3 páginas), porém apresentando um número alarmante de tentativas de login falhas (5, 9 ou 10 tentativas).

# Todas as anomalias identificadas representam um possível problema de segurança? Justifique.

Não. Nem toda anomalia estatística equivale a um incidente de segurança digital real.

O Perfil de Autenticação Suspeita (ex: Log 78, às 23h, vindo de um IP desconhecido com 10 tentativas de login consecutivas e desconexão rápida) possui fortes características técnicas de um ataque de força bruta automatizado ou tentativa de invasão de conta (credential stuffing). Este caso representa, sim, um risco imediato de segurança.

Por outro lado, o Perfil de Volume Atípico (ex: Log 54, às 23h, com duração de 5 horas e 110 páginas navegadas de forma bem-sucedida no primeiro login) pode ser perfeitamente uma atividade legítima de TI, como:

Um script de backup automatizado agendado pela própria empresa para rodar na madrugada.

Um processo de extração de relatórios consolidados de fechamento mensal executado por uma API autorizada.

Um funcionário de infraestrutura realizando manutenção extraordinária de sistemas fora do horário comercial.

# Que tipos de falso positivo podem ocorrer nesse cenário?

O modelo pode disparar alertas falsos (Falsos Positivos) em situações cotidianas da operação, tais como:

O usuário esquecido: Um funcionário legítimo em regime de home office ou viagem internacional (gerando IP atípico e horário avançado) erra sua senha corporativa 5 vezes seguidas antes de lembrar a combinação correta e acessar o painel. Ele agirá como o perfil de força bruta, mas é um acesso autorizado.

Auditorias de emergência: Um diretor financeiro acessa o sistema de madrugada para extrair dezenas de contratos para responder a uma fiscalização urgente, gerando uma sessão longa com volumetria de páginas alta.

# Como esse tipo de modelo poderia ser usado em um sistema real de monitoramento?

Em um ambiente corporativo de produção, este modelo de Machine Learning atuaria integrado a uma infraestrutura de SIEM (Security Information and Event Management) através de três pilares:

MFA Dinâmico e Adaptativo: Em vez de exigir dupla autenticação (MFA) em todos os cliques da empresa (o que estressa o usuário), o sistema monitora o score do Isolation Forest em tempo real. Se o comportamento de navegação de um funcionário começar a decair para a zona de anomalia, o sistema exige uma validação biométrica ou token SMS para autorizar a continuidade da sessão.

Geração de Alertas Priorizados para o SOC (Security Operations Center): O modelo filtra o ruído de milhares de logs normais diários e envia para a tela dos analistas de segurança apenas os registros com score negativo, aumentando a eficiência do time de monitoramento.

Gatilhos de Resposta Automatizada: Diante de anomalias críticas (como o pico de 10 tentativas de login na madrugada sob IP suspeito), o sistema pode disparar um gatilho de API bloqueando o IP na camada do Firewall e suspendendo temporariamente a conta do usuário por 30 minutos até uma análise humana.

Equil-brio-da-Terra-
Fala sobre a sustentabilidade e o plantio de árvores 
// Jogo: Agro Sustentável - Equilíbrio da Terra
// Tema: Agro forte, futuro sustentável
// Para p5.js

let solo = [];
let arvores = [];
let cultivos = [];
let agua = [];
let poluicao = 0;
let producao = 0;
let biodiversidade = 100;
let pontuacao = 0;
let fase = 1;
let mensagem = "";
let tempoMensagem = 0;

// Cores sustentáveis
let corSolo = [101, 67, 33];
let corAgua = [64, 164, 255];
let corArvore = [34, 139, 34];
let corCultivo = [255, 215, 0];
let corPoluicao = [128, 128, 128];

function setup() {
  createCanvas(1200, 700);
  textAlign(CENTER, CENTER);
  textSize(16);
  
  // Inicializar elementos do solo
  for(let i = 0; i < 20; i++) {
    solo.push({
      x: random(width),
      y: random(height - 100, height - 50),
      fertilidade: random(0.5, 1)
    });
  }
  
  // Adicionar água inicial
  for(let i = 0; i < 5; i++) {
    agua.push({
      x: random(width),
      y: random(height - 150, height - 80),
      quantidade: random(30, 50)
    });
  }
  
  // Adicionar algumas árvores iniciais
  for(let i = 0; i < 3; i++) {
    adicionarArvore(random(width), random(height - 200, height - 100));
  }
}

function draw() {
  // Fundo gradiente (céu)
  for(let i = 0; i <= height; i++) {
    let inter = map(i, 0, height, 0, 1);
    let c = lerpColor(color(135, 206, 235), color(173, 216, 230), inter);
    stroke(c);
    line(0, i, width, i);
  }
  
  // Desenhar solo
  fill(corSolo[0], corSolo[1], corSolo[2]);
  noStroke();
  rect(0, height - 100, width, 100);
  
  // Desenhar solo fértil (manchas)
  for(let s of solo) {
    fill(139, 69, 19, 100);
    ellipse(s.x, s.y, 30, 10);
    fill(34, 139, 34, s.fertilidade * 100);
    ellipse(s.x, s.y - 5, 20, 8);
  }
  
  // Desenhar água
  for(let a of agua) {
    fill(corAgua[0], corAgua[1], corAgua[2], 150);
    ellipse(a.x, a.y, a.quantidade, 20);
    fill(100, 200, 255, 100);
    ellipse(a.x, a.y - 5, a.quantidade * 0.8, 15);
  }
  
  // Desenhar árvores
  for(let arv of arvores) {
    // Tronco
    fill(101, 67, 33);
    rect(arv.x - 5, arv.y, 10, 30);
    // Copa
    fill(corArvore[0], corArvore[1], corArvore[2]);
    ellipse(arv.x, arv.y - 5, 25, 25);
    ellipse(arv.x - 8, arv.y, 20, 20);
    ellipse(arv.x + 8, arv.y, 20, 20);
    
    // Frutos (se produção > 50)
    if(producao > 50) {
      fill(255, 69, 0);
      ellipse(arv.x, arv.y - 8, 5, 5);
    }
  }
  
  // Desenhar cultivos
  for(let cult of cultivos) {
    fill(corCultivo[0], corCultivo[1], corCultivo[2]);
    rect(cult.x - 3, cult.y - 15, 6, 15);
    fill(50, 205, 50);
    ellipse(cult.x, cult.y - 18, 10, 8);
    
    // Mostrar estágio de crescimento
    fill(0);
    textSize(12);
    text("🌱", cult.x, cult.y - 25);
  }
  
  // Desenhar poluição
  if(poluicao > 0) {
    fill(corPoluicao[0], corPoluicao[1], corPoluicao[2], poluicao * 2);
    for(let i = 0; i < poluicao / 10; i++) {
      ellipse(random(width), random(height - 100, height), random(10, 30), random(5, 15));
    }
  }
  
  // Interface do jogador
  desenharInterface();
  
  // Atualizar dinâmicas do sistema
  atualizarSistema();
  
  // Mostrar mensagem temporária
  if(millis() < tempoMensagem) {
    fill(255, 215, 0);
    stroke(0);
    strokeWeight(2);
    textSize(20);
    text(mensagem, width/2, 100);
  }
  
  // Verificar condições de vitória/derrota
  verificarCondicoes();
}

function desenharInterface() {
  fill(255, 255, 255, 200);
  rect(10, 10, 200, 150, 10);
  fill(0);
  textSize(14);
  textAlign(LEFT, TOP);
  text(`🌾 Produção: ${Math.floor(producao)}`, 20, 20);
  text(`🌳 Biodiversidade: ${Math.floor(biodiversidade)}%`, 20, 45);
  text(`⚠️ Poluição: ${Math.floor(poluicao)}%`, 20, 70);
  text(`⭐ Pontuação: ${pontuacao}`, 20, 95);
  text(`📊 Fase: ${fase}`, 20, 120);
  
  // Barra de equilíbrio
  fill(255);
  rect(width - 220, 20, 200, 30);
  let equilibrio = (biodiversidade - poluicao + 100) / 2;
  fill(0, 255, 0);
  rect(width - 220, 20, equilibrio * 2, 30);
  fill(255, 0, 0);
  rect(width - 220 + equilibrio * 2, 20, (100 - equilibrio) * 2, 30);
  fill(0);
  textSize(12);
  text("Equilíbrio", width - 120, 40);
  
  // Instruções
  fill(255, 255, 255, 200);
  rect(width - 310, height - 60, 300, 50, 5);
  fill(0);
  textSize(12);
  textAlign(CENTER);
  text("Pressione: [1] Plantar Árvore | [2] Plantar Cultivo | [3] Irrigar | [R] Reiniciar", width - 160, height - 35);
}

function atualizarSistema() {
  // Produção aumenta com cultivos e diminui com poluição
  let producaoBase = cultivos.length * 5;
  let penalidadePoluicao = poluicao * 0.5;
  producao = max(0, producaoBase - penalidadePoluicao);
  
  // Biodiversidade aumenta com árvores e diminui com poluição
  let bioBase = arvores.length * 8;
  let bioPerda = poluicao * 1.2;
  biodiversidade = constrain(bioBase - bioPerda, 0, 100);
  
  // Poluição diminui gradualmente com árvores e água
  let reducaoPoluicao = (arvores.length * 0.5) + (agua.length * 0.3);
  poluicao = max(0, poluicao - reducaoPoluicao);
  
  // Pontuação baseada no equilíbrio
  pontuacao = floor(producao * (biodiversidade / 100) * fase);
  
  // Cultivos podem morrer com poluição alta
  if(poluicao > 70 && random() < 0.02) {
    if(cultivos.length > 0) {
      cultivos.pop();
      mensagem = "⚠️ Cultivos morreram pela poluição!";
      tempoMensagem = millis() + 2000;
    }
  }
  
  // Árvores podem morrer com poluição extrema
  if(poluicao > 85 && random() < 0.01) {
    if(arvores.length > 0) {
      arvores.pop();
      mensagem = "😢 Uma árvore morreu pela poluição!";
      tempoMensagem = millis() + 2000;
    }
  }
}

function adicionarArvore(x, y) {
  if(arvores.length < 30 && y > height - 150 && y < height - 50) {
    arvores.push({x: x, y: y});
    mensagem = "🌳 Árvore plantada! Aumenta a biodiversidade!";
    tempoMensagem = millis() + 1500;
  }
}

function adicionarCultivo(x, y) {
  if(cultivos.length < 40 && y > height - 130 && y < height - 60) {
    cultivos.push({x: x, y: y});
    mensagem = "🌾 Cultivo plantado! Aumenta a produção!";
    tempoMensagem = millis() + 1500;
  }
}

function adicionarAgua(x, y) {
  if(agua.length < 15) {
    agua.push({x: x, y: y, quantidade: 40});
    mensagem = "💧 Água adicionada! Reduz a poluição!";
    tempoMensagem = millis() + 1500;
  }
}

function keyPressed() {
  if(key === '1') {
    adicionarArvore(mouseX, mouseY);
  } else if(key === '2') {
    adicionarCultivo(mouseX, mouseY);
  } else if(key === '3') {
    adicionarAgua(mouseX, mouseY);
  } else if(key === 'r' || key === 'R') {
    reiniciarJogo();
  }
}

function mousePressed() {
  // Instrução visual
  mensagem = "Use as teclas 1, 2, 3 para interagir!";
  tempoMensagem = millis() + 1500;
}

function verificarCondicoes() {
  // Vitória por equilíbrio sustentável
  if(biodiversidade > 80 && producao > 100 && poluicao < 20) {
    fill(255, 215, 0);
    stroke(0);
    strokeWeight(3);
    textSize(32);
    text("🏆 VITÓRIA! 🏆\nAgro Forte e Sustentável!", width/2, height/2);
    textSize(16);
    text("Parabéns! Você alcançou o equilíbrio perfeito!", width/2, height/2 + 50);
    noLoop();
  }
  
  // Derrota por poluição excessiva
  if(poluicao > 90) {
    fill(128, 128, 128);
    stroke(0);
    strokeWeight(3);
    textSize(32);
    text("💀 DERROTA! 💀\nPoluição destruiu o ecossistema!", width/2, height/2);
    textSize(16);
    text("Pressione R para recomeçar", width/2, height/2 + 50);
  }
  
  // Avançar de fase
  if(pontuacao > 500 * fase && fase < 5) {
    fase++;
    mensagem = `🎉 FASE ${fase} alcançada! Desafios aumentam! 🎉`;
    tempoMensagem = millis() + 3000;
    
    // Aumentar desafio
    poluicao += 10;
    if(fase >= 3) {
      // Nova mecânica: pragas nos cultivos
      if(cultivos.length > 0 && random() < 0.3) {
        cultivos.splice(floor(random(cultivos.length)), 1);
      }
    }
  }
}

function reiniciarJogo() {
  solo = [];
  arvores = [];
  cultivos = [];
  agua = [];
  poluicao = 0;
  producao = 0;
  biodiversidade = 100;
  pontuacao = 0;
  fase = 1;
  
  // Reinicializar elementos
  for(let i = 0; i < 20; i++) {
    solo.push({
      x: random(width),
      y: random(height - 100, height - 50),
      fertilidade: random(0.5, 1)
    });
  }
  
  for(let i = 0; i < 5; i++) {
    agua.push({
      x: random(width),
      y: random(height - 150, height - 80),
      quantidade: random(30, 50)
    });
  }
  
  for(let i = 0; i < 3; i++) {
    adicionarArvore(random(width), random(height - 200, height - 100));
  }
  
  loop();
  mensagem = "Jogo reiniciado! Busque o equilíbrio sustentável!";
  tempoMensagem = millis() + 2000;
}

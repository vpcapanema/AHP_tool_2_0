# SIGMA AHP - Upload de Matriz de Comparação Pareada

## 📋 Visão Geral

A funcionalidade de **Upload de Matriz** permite que usuários carreguem uma matriz de comparação pareada já preenchida, pulando as etapas de preenchimento manual e indo direto para os resultados.

## 🔄 Como Funciona

### Passo 1: Escolha do Método
No **Step 1**, o usuário pode escolher entre:
- **Método Manual**: Processo guiado passo a passo (steps 2-5)
- **Upload de Matriz**: Upload de arquivo pré-preenchido (pula para step 5)

### Passo 2: Download do Modelo
O sistema oferece dois formatos de template:

#### CSV (exemplo_matriz_ahp.csv)
```csv
,Custo,Qualidade,Tempo,Risco
Custo,1,0.333,0.5,2
Qualidade,3,1,2,5
Tempo,2,0.5,1,3
Risco,0.5,0.2,0.333,1
```

#### JSON (exemplo_matriz_ahp.json)
```json
{
  "criteria": ["Custo", "Qualidade", "Tempo", "Risco"],
  "matrix": [
    [1, 0.333, 0.5, 2],
    [3, 1, 2, 5],
    [2, 0.5, 1, 3],
    [0.5, 0.2, 0.333, 1]
  ]
}
```

### Passo 3: Preenchimento
O usuário edita o arquivo baixado com suas comparações usando a **Escala de Saaty**:

| Valor | Significado |
|-------|-------------|
| 1 | Igual importância |
| 3 | Importância moderada |
| 5 | Importância forte |
| 7 | Importância muito forte |
| 9 | Importância extrema |
| 2, 4, 6, 8 | Valores intermediários |

**Importante:**
- Para valores fracionários (1/3, 1/5, etc.), use decimais (0.333, 0.2)
- A diagonal principal deve sempre ser 1
- A matriz deve ser recíproca: se A→B = 3, então B→A = 1/3 (0.333)

### Passo 4: Upload e Processamento
- Usuário faz upload do arquivo preenchido (.csv ou .json)
- Sistema valida:
  - Formato do arquivo
  - Matriz quadrada (n×n)
  - Diagonal principal = 1
  - Reciprocidade (com tolerância de 0.01)
- Se válido, redireciona para **Step 5 (Resultados)**

## 🔍 Validações Implementadas

### Validação de Formato
- **CSV**: Verifica estrutura de matriz com cabeçalhos
- **JSON**: Valida propriedades "criteria" e "matrix"

### Validação de Conteúdo
```javascript
// Diagonal principal deve ser 1
matrix[i][i] === 1

// Reciprocidade
matrix[i][j] ≈ 1 / matrix[j][i]  (tolerância: 0.01)

// Valores numéricos válidos
!isNaN(parseFloat(value))

// Matriz quadrada
matrix.length === criteria.length
matrix[i].length === criteria.length
```

### Mensagens de Erro
O sistema fornece mensagens específicas para cada tipo de erro:
- "Arquivo CSV inválido: muito poucas linhas"
- "Valor inválido na linha X, coluna Y"
- "Matriz não é quadrada. Critérios: X, Linhas: Y"
- "Diagonal principal inválida na posição [i,j]"
- "Reciprocidade não exata em [i,j]" (warning apenas)

## 💾 Armazenamento de Dados

### localStorage Keys
```javascript
ahp_inputMethod: 'upload' | 'manual'
ahp_criteriaCount: number
ahp_criteria: JSON.stringify(string[])
ahp_uploadedMatrix: JSON.stringify(number[][])
ahp_pairwiseMatrix: JSON.stringify(number[][])
```

### Fluxo de Dados
1. **Upload**: `ahp_uploadedMatrix` armazena matriz original
2. **Step5**: Detecta `ahp_inputMethod === 'upload'`
3. **Processamento**: Usa `ahp_uploadedMatrix` para cálculos
4. **Compatibilidade**: Copia para `ahp_pairwiseMatrix` (formato padrão)

## 🎨 Interface do Usuário

### Componentes Visuais
- **Radio buttons**: Seleção de método (manual vs upload)
- **Cards expansíveis**: Mostram conteúdo baseado na seleção
- **Badges numerados**: Indicam passos do processo (1, 2, 3)
- **Info boxes**: Instruções com ícones e formatação destacada
- **Upload area**: Área com drag-and-drop visual
- **File info display**: Mostra arquivo selecionado com ícone e tamanho
- **Notifications**: Toast messages para feedback (success/error/info)

### Animações
```css
@keyframes slideIn {
  from { transform: translateX(400px); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
```

### Responsividade
- Grid adaptativo para botões de template
- Cards flexíveis para diferentes tamanhos de tela
- Notificações fixas no canto superior direito

## 🔧 Funções JavaScript Principais

### downloadMatrixTemplate(format)
Gera e faz download de arquivo modelo (CSV ou JSON)

### handleFileSelect(event)
Processa seleção de arquivo pelo usuário

### parseCSVMatrix(content)
Converte conteúdo CSV em objeto { criteria, matrix }

### parseJSONMatrix(content)
Valida e extrai dados de JSON

### validateMatrix(criteria, matrix)
Valida regras de AHP (diagonal, reciprocidade)

### processUploadedMatrix()
Salva dados e redireciona para resultados

### showNotification(message, type)
Exibe toast notification temporária

## 📊 Exemplo Completo

### Cenário de Uso
**Problema**: Escolher fornecedor baseado em 4 critérios

**Critérios**:
- Custo
- Qualidade
- Tempo de entrega
- Risco

**Comparações** (Custo vs outros):
- Custo vs Qualidade: Qualidade é 3× mais importante → valor = 0.333
- Custo vs Tempo: Tempo é 2× mais importante → valor = 0.5
- Custo vs Risco: Custo é 2× mais importante → valor = 2

**Matriz Resultante**:
```
       Custo  Qualidade  Tempo  Risco
Custo    1      0.333    0.5     2
Qual.    3        1       2      5
Tempo    2      0.5       1      3
Risco   0.5     0.2     0.333    1
```

**Resultado Esperado** (pesos aproximados):
- Qualidade: ~48%
- Tempo: ~28%
- Custo: ~14%
- Risco: ~10%

## 🐛 Tratamento de Erros

### Arquivo Não Selecionado
```javascript
if (!uploadedMatrixData) {
    showNotification('Por favor, selecione um arquivo de matriz primeiro.', 'error');
    return;
}
```

### Parse Error
```javascript
try {
    uploadedMatrixData = parseCSVMatrix(content);
} catch (error) {
    showNotification('Erro ao ler arquivo: ' + error.message, 'error');
}
```

### Redirecionamento de Segurança
```javascript
if (criteria.length === 0 || uploadedMatrix.length === 0) {
    alert('Dados de upload incompletos. Redirecionando...');
    window.location.href = 'step1-criterios.html';
}
```

## 📝 Notas Técnicas

### Formatos Aceitos
- `.csv` (text/csv)
- `.json` (application/json)

### Limitações
- Máximo de 20 critérios (limitação geral do sistema)
- Tamanho de arquivo: sem limite explícito (navegador gerencia)
- Precisão numérica: 4 casas decimais para exibição

### Compatibilidade
- Funciona com todos os navegadores modernos (FileReader API)
- Não requer bibliotecas externas
- Usa apenas JavaScript vanilla + localStorage

### Performance
- Validação ocorre no cliente (sem servidor)
- Processamento instantâneo para até 20 critérios
- Uso mínimo de memória

## 🔄 Integração com Sistema Existente

A funcionalidade de upload é **completamente compatível** com o fluxo manual:

1. **Step1**: Bifurcação manual/upload
2. **Step2-4**: Ignorados se upload
3. **Step5**: Detecta origem automaticamente
4. **Exportação**: Funciona igualmente para ambos métodos

Não há diferença no resultado final - apenas no processo de entrada de dados.

## ✅ Checklist de Teste

- [ ] Download de template CSV funciona
- [ ] Download de template JSON funciona
- [ ] Upload de CSV válido redireciona para resultados
- [ ] Upload de JSON válido redireciona para resultados
- [ ] Validação detecta diagonal principal inválida
- [ ] Validação detecta matriz não-quadrada
- [ ] Validação detecta valores não-numéricos
- [ ] Validação avisa sobre reciprocidade inexata
- [ ] Notificações aparecem e desaparecem
- [ ] Resultados calculados estão corretos
- [ ] Exportação funciona após upload
- [ ] localStorage é limpo corretamente entre métodos
- [ ] Redirecionamento de erro funciona
- [ ] Interface responsiva em diferentes telas

## 🎯 Benefícios da Funcionalidade

1. **Velocidade**: Pula 3 etapas do processo manual
2. **Reutilização**: Permite carregar matrizes salvas anteriormente
3. **Integração**: Aceita matrizes de outras ferramentas (Excel, R, Python)
4. **Validação**: Garante consistência dos dados antes do processamento
5. **Flexibilidade**: Suporta múltiplos formatos de arquivo
6. **Experiência**: Interface intuitiva com feedback visual

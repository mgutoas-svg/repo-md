# Tipagem Estrutural vs. Tipagem Nominal em TypeScript

## Resumo Essencial

O TypeScript adota nativamente um sistema de tipagem estrutural (*structural typing*), fundamentado no princípio conceitual do *duck typing* ("se caminha como um pato e grasna como um pato, então é um pato") [485, 497, 499]. Nesse modelo, a compatibilidade e a equivalência entre dois tipos não são determinadas pelos seus nomes ou por declarações explícitas de herança, mas sim pela sua estrutura interna e pela forma de suas propriedades e métodos [28, 499]. Se um objeto possui os membros requeridos por uma interface ou tipo esperado, o compilador do TypeScript o aceita como válido, permitindo alta flexibilidade e reduzindo o acoplamento entre módulos [28, 321, 499].

Apesar dessa flexibilidade ser ideal para o ecossistema dinâmico do JavaScript, ela pode apresentar desafios em cenários onde tipos com a mesma estrutura possuem significados semânticos totalmente distintos — como diferenciar uma string não sanitizada de uma string validada, ou um ID de usuário de um ID de produto [19, 485]. Para contornar essa limitação sem renunciar à checagem estática, a comunidade e o ecossistema utilizam técnicas de simulação de tipagem nominal, tais como *branded types* (tagging nominal com interseções) ou o uso de propriedades privadas em classes [18, 485].

## Conceitos & Frameworks

- **Tipagem Estrutural (*Structural Typing*)**: Sistema onde a compatibilidade de tipos é avaliada exclusivamente pelo formato, chaves e tipos dos membros presentes no objeto, ignorando o nome do tipo ou classe de origem [17, 28, 499].
- **Tipagem Nominal (*Nominal Typing*)**: Sistema em que a equivalência exige nomes explícitos idênticos ou relacionamentos declarados em uma hierarquia de classes/interfaces [17, 485].
- ***Duck Typing***: Conceito em tempo de execução advindo do JavaScript onde a presença de métodos e propriedades determina a capacidade de uso de um objeto [497, 499].
- ***Branded Types / Nominal Tagging***: Técnica avançada que utiliza interseções de tipos primários com objetos contendo propriedades discriminatórias únicas (ex: `__brand`) para forçar distinção nominal no compilador [485].
- **Uniões Discriminadas (*Discriminated Unions*)**: Padrão estrutural onde interfaces compartilham um membro literal comum (ex: `type` ou `kind`), permitindo ao compilador realizar o estreitamento seguro de tipos (*type narrowing*) [47, 108, 109].

### Framework Principal

**Mecanismo de Checagem Estrutural e Guardrails Determinísticos do TypeScript**

O sistema de tipos do TypeScript opera em tempo de compilação e é completamente eliminado na geração do código JavaScript final (*type erasure*) [4, 8, 307, 316]. Seus componentes de verificação funcionam da seguinte forma:

1. **Inspecao de Forma (*Shape Inspection*)**: Ao passar um argumento para uma função, o compilador verifica se o valor possui todas as propriedades requeridas com os tipos correspondentes [28, 321]. Propriedades extras são permitidas em atribuições indiretas [28].
2. **Exceção Nominal por Campos Privados**: Se duas classes possuem a mesma estrutura pública, mas contêm um membro privado (`private`), o TypeScript passa a tratá-las de forma nominal, impedindo a atribuição cruzada entre instâncias de classes distintas [18].
3. **Guards de Tipo e Narrowing**: Utilitários de controle de fluxo (como `typeof`, operador `in` ou instruções `switch` em uniões discriminadas) informam o compilador sobre o tipo específico de uma variável dentro de um bloco condicional [47, 106, 108].
4. **Guardrails Determinísticos em IA**: Em fluxos de engenharia generativa e agentes de IA, a verificação estática do compilador atua como um validador autônomo; quando o LLM gera código com incompatibilidade estrutural, as mensagens de diagnóstico do `tsc` alimentam o contexto do modelo para autocorreção imediata antes da execução em tempo de runtime [34, 35].

## Processo Passo-a-Passo

1. **Definição de Interfaces e Contratos Estruturais**: Crie abstrações claras utilizando `interface` ou `type` indicando os campos e métodos estritamente necessários para a operação [5, 68].
2. **Implementação de Uniões Discriminadas**: Quando trabalhar com múltiplos estados ou variantes de dados, adicione um campo literal constante (ex: `type: 'human' | 'horse'`) para permitir um estreitamento limpo via `switch` [108, 109].
3. **Criação de Branded Types para Segurança Semântica**: Para dados sensíveis (tokens, chaves, moedas, strings validadas), defina uma interseção entre o tipo primitivo e um tipo com tag única para impedir atribuição acidental de dados não validados [485].
4. **Habilitação de Flags Estritas no Compiler Options (`tsconfig.json`)**: Configure `"strict": true` e `"strictNullChecks": true` para garantir que incompatibilidades nulas ou de tipos implícitos sejam bloqueadas estritamente no *build* [16, 17, 286].
5. **Compilação e Remoção de Tipos (*Type Erasure*)**: Execute o compilador `tsc` para verificar o projeto e emitir o código JavaScript limpo e otimizado para produção [4, 59, 71, 307].

## Métricas & KPIs

- **Redução de Exceções em Runtime (`TypeError`)**: Percentual de erros de incompatibilidade de tipos capturados em tempo de compilação em comparação a falhas durante a execução [34, 305].
- **Índice de Estreitamento Seguro (*Type Safety Coverage*)**: Porcentagem do código utilizando estreitamentos de tipo explícitos (*type guards*) sem o uso de castings inseguros ou `any` implícito [16, 106, 286].
- **Desempenho de Build e Zero Runtime Overhead**: Garantia de que 100% da verificação nominal/estrutural ocorre no ambiente do compilador, gerando overhead zero de performance no código JavaScript compilado [4, 8, 316].
- **Taxa de Autocorreção de Código por Agentes de IA**: Porcentagem de correções bem-sucedidas realizadas autonomamente por modelos generativos orientados pelas diagnósticas de tipo do `tsserver` [34, 35].

## Templates & Exemplos

```typescript
// 1. Tipagem Estrutural Clássica
interface Vehicle {
    make: string;
    model: string;
    year: number;
}

function displayVehicle(vehicle: Vehicle): void {
    console.log(`${vehicle.year} ${vehicle.make} ${vehicle.model}`);
}

// Objeto literal sem declaração explícita da interface Vehicle
const myCar = { make: "Toyota", model: "Corolla", year: 2023, color: "Silver" };
displayVehicle(myCar); // OK! Estruturalmente compatível.

// 2. Uniões Discriminadas para Estreitamento Seguro (Type Narrowing)
interface Human {
    type: 'human';
    walkingSpeed: number;
}

interface Horse {
    type: 'horse';
    runningSpeed: number;
}

type Mammal = Human | Horse;

function moveMammal(mammal: Mammal): void {
    switch (mammal.type) {
        case 'human':
            console.log(`Humano caminhando a ${mammal.walkingSpeed} km/h`);
            break;
        case 'horse':
            console.log(`Cavalo correndo a ${mammal.runningSpeed} km/h`);
            break;
    }
}

// 3. Simulação de Tipagem Nominal (Branded Types / Tagging)
export type Brand<K, T> = K & { readonly __brand: T };

export type UserId = Brand<string, "UserId">;
export type ProductId = Brand<string, "ProductId">;

function makeUserId(id: string): UserId {
    return id as UserId;
}

function makeProductId(id: string): ProductId {
    return id as ProductId;
}

function getUser(id: UserId) {
    console.log(`Buscando usuário: ${id}`);
}

const uId = makeUserId("usr_12345");
const pId = makeProductId("prd_98765");

getUser(uId); // OK
// getUser(pId); // ERRO DE COMPILAÇÃO: ProductId não é atribuível a UserId
```

## Aprendizados & Casos

- **Injeção de Comportamento sem Acoplamento**: A tipagem estrutural permite abstrair comportamentos de código legado ou bibliotecas de terceiros sem a necessidade de alterar a declaração original da classe, bastando definir uma interface com os membros requeridos [21, 321].
- **Diferenciação por Membros Privados**: Ao trabalhar com Orientação a Objetos no TypeScript, a presença de propriedades privadas em classes cria um comportamento nominal implícito, impedindo que instâncias de classes idênticas em estrutura pública sejam trocadas inadvertidamente [18].
- **Suporte Determinístico em Sistemas Autônomos**: A estrutura do sistema de tipos fornece o contexto ideal para o Model Context Protocol (MCP) e ferramentas de desenvolvimento assistidas por IA, agindo como um contrato determinístico inquebrável para agentes de código [34, 35].

## Integração

- Link → [typescript-tsconfig-guide.md]
- Link → [design-patterns-typescript.md]
- Link → [ai-generative-guardrails.md]

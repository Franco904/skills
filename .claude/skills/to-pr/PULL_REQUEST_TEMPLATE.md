- closes #xyz
- fixes #xyz

## 1. Análise de Impactos

Este PR altera algum módulo compartilhado ou estabilizado?  

<details>
<summary>Exemplos</summary>
</br>
  
1. Widget/componente compartilhado
2. Tema global / estilos compartilhados
3. Arquivos do módulo comum / utilitários compartilhados
4. Classe, serviço ou helper usado por múltiplos módulos
5. Navegação / rotas compartilhadas
6. Estado global / providers / stores / controllers compartilhados
7. Estrutura de banco / migrations / queries
8. Contrato de API / serialização / parsing
9. Permissões / autenticação / sessão

Obs.: Qualquer definição de arquitetura ou arquivo em comum que possa impactar outras partes do aplicativo deve ser considerada. 

</details>

**Se sim, informe:**
- [ ] 1. Quais alterações foram necessárias?
- [ ] 2. Por que foi necessário realizar essa alteração?
- [ ] 3. A solução adotada foi resultado de uma troca real? 

## 2. Orientações para Review

Informe aqui qualquer ponto que mereça atenção extra durante a revisão.</br>
Caso algum item além dos critérios de aceite da tarefa tenha sido alterado, isso deve ser sinalizado aqui. Seja conciso na descrição.

## 3. Orientações para QA

Informe aqui qualquer ponto que mereça atenção extra durante o teste.</br>
Caso algum item além dos critérios de aceite da tarefa tenha sido alterado, isso deve ser sinalizado aqui.

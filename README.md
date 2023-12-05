# Access List Control - Conceitos e exemplos

    Ao configurar uma ACL complexa, sugere-se que você:
    Use um editor de texto e escreva as especificidades da política a ser implementada.
    Adicione os comandos de configuração do IOS para realizar essas tarefas.
    Incluir observações para documentar a ACL.
    Copie e cole os comandos no dispositivo.
    Sempre teste minuciosamente uma ACL para garantir que ela aplica corretamente a política desejada.

    # Sintaxe numerada da ACL IPv4 padrão
        Router(config)# access-list access-list-number {deny | permit | remark text} source [source-wildcard] [log]

            Parâmetro	                    Descrição
            access-list-number.............Este é o número decimal da ACL. O intervalo de números ACL padrão é de 1 a 99 ou 1300 a 1999.
            deny...........................Nega o acesso se as condições corresponderem.
            permit.........................Permite o acesso se as condições corresponderem.
            remark text....................(Opcional) Isso adiciona uma entrada de texto para fins de documentação. Cada observação é limitada a 100 caracteres.
            source.........................Isso identifica a rede de origem ou o endereço do host a ser filtrado.
                                           Use a palavra-chave any para especificar todas as redes.
                                           Use a palavra-chave host ip address ou simplesmente digite um endereço IP (sem o host ) para identificar um endereço IP específico.
            source-wildcard................(Opcional) Máscara curinga de 32 bits a ser aplicada à origem. . Se omitido, uma máscara padrão 0.0.0.0 é assumida.
            log............................(Opcional) Esta palavra-chave gera e envia uma mensagem informativa sempre que a ACE for correspondida.
                                            A mensagem inclui número ACL, condição correspondente (isto é, permitido ou negado), endereço de origem e número de pacotes.
                                            Esta mensagem é gerada para o primeiro pacote correspondente.
                                            Esta palavra-chave só deve ser implementada para resolução de problemas ou motivos de segurança.
                        
    
    # Sintaxe nomeada da ACL IPv4 padrão
        Router(config)# ip access-list standard access-list-name

        Observação: Use o comando no ip access-list standard access-list-name global configuration para remover uma ACL IPv4 padrão nomeada.


    
    # Aplicar uma ACL IPv4 padrão
        Depois que uma ACL IPv4 padrão é configurada, ela deve ser vinculada a uma interface ou recurso. O comando a seguir pode ser usado para vincular uma ACL IPv4 padrão numerada ou nomeada a uma interface:

            Router(config-if) # ip access-group {access-list-number | access-list-name} {in | out}

            Obs.: Para remover uma ACL de uma interface, primeiro digite o comando no ip access-group interface configuration. No entanto, a ACL ainda será configurada no roteador.  
                  Para remover a ACL do roteador, use o comando de configuração no access-list global.

             ![Alt text](image.png)     

             Suponha que apenas PC1 é permitido sair para a internet. Para habilitar essa diretiva, uma ACL ACE padrão pode ser aplicada de saída em S0/1/0, conforme mostrado na figura.
                
                R1(config)# access-list 10 remark ACE permits ONLY host 192.168.10.10 to the internet
                R1(config)# access-list 10 permit host 192.168.10.10
                R1(config)# do show access-lists
                    Standard IP access list 10
                        10 permit 192.168.10.10
                R1(config)#

                Observe que a saída do show access-lists comando não exibe as remark instruções. As observações da ACL são exibidas no arquivo de configuração em execução. Embora o remark comando não seja necessário para habilitar a ACL, ele é fortemente sugerido para fins de documentação.

             Agora suponha que uma nova diretiva de rede afirma que os hosts na LAN 2 também devem ter permissão para a Internet. Para habilitar essa diretiva, uma segunda ACL ACE padrão pode ser adicionada à ACL 10, conforme mostrado na saída.

                R1(config)# access-list 10 remark ACE permits all host in LAN 2
                R1(config)# access-list 10 permit 192.168.20.0 0.0.0.255
                R1(config)# do show access-lists
                Standard IP access list 10
                    10 permit 192.168.10.10
                    20 permit 192.168.20.0, wildcard bits 0.0.0.255
                R1(config)#   

             Aplique a saída da ACL 10 na interface Serial 0/1/0.
                R1(config)# interface Serial 0/1/0
                R1(config-if)# ip access-group 10 out
                R1(config-if)# end
                R1#   

             A política resultante da ACL 10 só permitirá que o host 192.168.10.10 e todos os hosts da LAN 2 saiam da interface Serial 0/1/0. Todos os outros hosts na rede 192.168.10.0 não serão permitidos na Internet.

             Use o show running-config comando para revisar a ACL na configuração, conforme mostrado na saída.

                R1# show run | section access-list
                access-list 10 remark ACE permits host 192.168.10.10
                access-list 10 permit 192.168.10.10
                access-list 10 remark ACE permits all host in LAN 2
                access-list 10 permit 192.168.20.0 0.0.0.255
                R1#

             Finalmente, use o show ip interface comando para verificar se uma interface tem uma ACL aplicada a ela. Na saída de exemplo, a saída está especificamente olhando para a interface Serial 0/1/0 para linhas que incluem texto de “lista de acesso”.

                R1# show ip int Serial 0/1/0 | include access list
                Outgoing Common access list is not set
                Outgoing access list is 10
                Inbound Common access list is not set
                Inbound  access list is not set
                R1#    

     
    # Exemplo de ACL IPv4 padrão nomeado

              Suponha que apenas PC1 é permitido sair para a internet. Para habilitar essa diretiva, uma ACL padrão nomeada chamada Permit-ACCESS poderia ser aplicada de saída em S0/1/0.

                Remova a ACL 10 configurada anteriormente e crie uma ACL padrão nomeada chamada Permit-ACCESS, conforme mostrado aqui.

                R1(config)# no access-list 10
                R1(config)# ip access-list standard PERMIT-ACCESS
                R1(config-std-nacl)# remark ACE permits host 192.168.10.10
                R1(config-std-nacl)# permit host 192.168.10.10
                R1(config-std-nacl)#

    
             Agora adicione uma ACE permitindo apenas host 192.168.10.10 e outra ACE permitindo todos os hosts LAN 2 para a internet.

                R1(config-std-nacl)# remark ACE permits host 192.168.10.10
                R1(config-std-nacl)# permit host 192.168.10.10
                R1(config-std-nacl)# remark ACE permits all hosts in LAN 2
                R1(config-std-nacl)# permit 192.168.20.0 0.0.0.255
                R1(config-std-nacl)# exit
                R1(config)#

            Aplique a nova saída ACL nomeada à interface Serial 0/1/0.

                R1(config)# interface Serial 0/1/0
                R1(config-if)# ip access-group PERMIT-ACCESS out
                R1(config-if)# end
                R1#

            
            Use o show access-lists comando show running-config e para revisar a ACL na configuração, conforme mostrado na saída.

                R1# show access-lists
                Standard IP access list PERMIT-ACCESS
                    10 permit 192.168.10.10
                    20 permit 192.168.20.0, wildcard bits 0.0.0.255
                R1# show run | section ip access-list
                ip access-list standard PERMIT-ACCESS
                remark ACE permits host 192.168.10.10
                permit 192.168.10.10
                remark ACE permits all hosts in LAN 2
                permit 192.168.20.0 0.0.0.255
                R1#

            Finalmente, use o show ip interface comando para verificar se uma interface tem uma ACL aplicada a ela. Na saída de exemplo, a saída está especificamente olhando para a interface Serial 0/1/0 para linhas que incluem texto de “lista de acesso”.

                R1# show ip int Serial 0/1/0 | include access list
                Outgoing Common access list is not set
                Outgoing access list is PERMIT-ACCESS
                Inbound Common access list is not set
                Inbound  access list is not set
                R1#


        - Exercício
            
            Crie uma ACL numerada que nega o host 192.168.10.10, mas permite que todos os outros hosts na LAN 1. Comece configurando a ACL 20 ACE que nega o host 192.168.10.10 usando a host palavra-chave.




    # Dois métodos para modificar uma ACL
        - Esta seção discutirá dois métodos a serem usados ao modificar uma ACL:

            >> Método do Editor de Texto
            >> Método dos números de sequência
        


        >> Método do Editor de Texto
            R1# show run | section access-list 
            access-list 1 deny 19.168.10.10
            access-list 1 permit 192.168.10.0 0.0.0.255

            Suponha que a ACL 1 foi corrigida. Portanto, a ACL incorreta deve ser excluída e as instruções ACL 1 corrigidas devem ser coladas no modo de configuração global, conforme mostrado na saída.

            R1(config)# no access-list 1
            R1(config)#
            R1(config)# access-list 1 deny 192.168.10.10
            R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255

         
        >> Método dos números de sequência
            Uma ACL ACE também pode ser excluída ou adicionada usando os números de sequência ACL. Os números de sequência são atribuídos automaticamente quando uma ACE é inserida. Estes números estão listados no show access-lists comando. O show running-config comando não exibe números de sequência.
                R1# show access-lists 
                Standard IP access list 1 
                    10 deny 19.168.10.10 
                    20 permit 192.168.10.0, wildcard bits 0.0.0.255
                R1#

            Use o ip access-list standard comando para editar uma ACL.
               R1# conf t
                R1(config)# ip access-list standard 1
                R1(config-std-nacl)# no 10
                R1(config-std-nacl)# 10 deny host 192.168.10.10
                R1(config-std-nacl)# end
                R1# show access-lists
                Standard IP access list 1
                    10 deny   192.168.10.10
                    20 permit 192.168.10.0, wildcard bits 0.0.0.255
                R1#



        >> Modificar um exemplo de ACL nomeado
            As ACLs nomeadas também podem usar números de sequência para excluir e adicionar ACEs. Consulte o exemplo para ACL NO-ACCESS.

            R1# show access-lists
            Standard IP access list NO-ACCESS
                10 deny   192.168.10.10
                20 permit 192.168.10.0, wildcard bits 0.0.0.255
            Suponha que o host 192.168.10.5 da rede 192.168.10.0/24 também deve ter sido negado. Se você inseriu uma nova ACE, ela será anexada ao final da ACL. Portanto, o host nunca seria negado porque o ACE 20 permite todos os hosts dessa rede.


            Observação: A função de hash é aplicada apenas a instruções de host em uma lista de acesso padrão IPv4. Os detalhes da função de hash estão fora do escopo deste curso.

                R1# configure terminal
                R1(config)# ip access-list standard NO-ACCESS
                R1(config-std-nacl)# 15 deny 192.168.10.5
                R1(config-std-nacl)# end
                R1#
                R1# show access-lists
                Standard IP access list NO-ACCESS
                    15 deny   192.168.10.5
                    10 deny   192.168.10.10
                    20 permit 192.168.10.0, wildcard bits 0.0.0.255
                R1#

        # Estatísticas de ACL
            Use o clear access-list counters comando para limpar as estatísticas de ACL. Esse comando pode ser utilizado sozinho ou com o número ou o nome de uma ACL específica.

                R1# show access-lists
                Standard IP access list NO-ACCESS
                    10 deny   192.168.10.10  (20 matches) 
                    20 permit 192.168.10.0, wildcard bits 0.0.0.255  (64 matches) 
                R1# clear access-list counters NO-ACCESS
                R1# show access-lists
                Standard IP access list NO-ACCESS
                    10 deny   192.168.10.10
                    20 permit 192.168.10.0, wildcard bits 0.0.0.255
                R1#



        # MODIFICANDO ACL
        
            Adicione duas linhas adicionais no final da ACL. No modo de configuração global, modifique ACL, BRANCH-OFFICE-POLICY.

                R1#(config)# ip access-list standard BRANCH-OFFICE-POLICY
                R1(config-std-nacl)# 30 permit 209.165.200.224 0.0.0.31
                R1(config-std-nacl)# 40 deny any
                R1(config-std-nacl)# end
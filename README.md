# dataset-cbo

A Classificação Brasileira de Ocupações (**CBO**), instituída pela [portaria ministerial nº. 397 de 2002](http://www.mtecbo.gov.br/cbosite/pages/legislacao.jsf), tem por finalidade a identificação das ocupações no mercado de trabalho, para fins classificatórios junto aos registros administrativos e domiciliares.

Ela se encontra disponível apenas em formato PDF, atualmente (2016) em [www.mtecbo.gov.br/cbosite](http://www.mtecbo.gov.br/cbosite/pages/download?tipoDownload=1) (no link "*downloads*").

O objetivo deste projeto é traduzir o PDF para um banco de dados aberto padronizado, que a princípio corresponde a um arquivo simples de planilha [formato CSV](https://pt.wikipedia.org/wiki/Comma-separated_values) com três colunas, relativas a

* código CBO: coluna `codigo`;
* termo associado ao código: coluna `termo`;
* tipo ou status do termo: "sinônimo", "classificação" ou "ocupação"; coluna `tipo`.

O resultado deste projeto poderá ser oferecido como *dataset* dentro do "ecosistema" [Data Packaged Core Datasets](http://data.okfn.org/roadmap/core-datasets) brasileiro. A proposta de *local datasets* foi lançada nesta discussão [_Country-specific data package register_](https://discuss.okfn.org/t/3178), e esta seria uma das primeiras contribuições dentro da proposta.

## Utilização das listas

Para apenas consultar termos e códigos oficiais canônicos, basta clicar [aqui em `data/lista_canonicos.csv`](https://github.com/okfn-brasil/dataset-cbo/blob/master/data/lista_canonicos.csv) para visualizar todas as ocupações:

* Para _download_: clique em "raw" (pode ser com o botão direito) para baixar a planilha. 
* Para buscas: ao lado do ícone de lupa (com mensagem "seach this file"), digitar trechos do termo procurado.

Para utilização em bancos de dados e terminologias, usar [`data/lista.csv`](https://github.com/okfn-brasil/dataset-cbo/blob/master/data/lista.csv), que é a lista completa, onde foram mantidos os sinônimos.

# Preparação
Procedimentos utilizados na preparação dos [dados deste projeto](data).  Todos os mataterais-fonte estão sendo mantidos na pasta `dumps`.
```sh
cd dumps

## COPIA:
wget -c --output-document=CBO2002_Liv1.pdf www.mtecbo.gov.br/cbosite/pages/download?tipoDownload=1
wget -c http://portalfat.mte.gov.br/wp-content/uploads/2016/02/CBO2002_LISTA.pdf

## CONVERSAO PRINCIPAL:
pdftotext -layout CBO2002_LISTA.pdf
cat CBO2002_LISTA.txt | grep -E "^[0-9]{4,}" | tr -s ',' ';' > tmp.txt
echo -e "codigo,termo,tipo" | cat - tmp.txt | sed  -r 's:     +:,:g' > ../data/lista.csv
rm tmp.txt # opcional, usar diff  para para conferir se não corrompeu no processo

## LISTA DERIVADA, sem sinônimos (só termos canônicos)
awk -F ","  \
 'BEGIN {print "codigo,termo"}  $3~/[Oo]cupa[ç][aã]o/ {print $1 "," $2}' \
 data/lista.csv > data/lista_canonicos.csv
```

O primeiro `wget` é apenas para a preservação do inteiro teor das definições CBO, que são fornecidas em PDF e portanto difíceis de serem extraídas na forma de lista. O nome de arquivo é o oficial, obtido via navegador.

Com o segundo `wget` se obtém uma listagem oficial (mte.gov.br) da lista desejada  &ndash; resta auditorar com outros recursos, como a lista de códigos apresentados no inteiro teor.

A opção `-layout` foi adotada por se mostrar mais adequada que a _default_ (`-raw`), recuperando linha a linha já num formato próximo ao de planilha. O `grep` garante a exclusão de cabeçalhos e rodapés, o `tr` evita eventuais ambiguidades com o separador CSV, e o comando `sed` faz o trabalho principal de conversão de formato, garantindo por fim uma lista estruturada.

Para a geração da lista canônica, sem sinônimos, foi utilizado um filtro simples [`awk`](https://en.wikipedia.org/wiki/AWK), eliminando a coluna `tipo` e mantendo apenas as linhas do tipo "ocupação".

NOTA: com o comando `cat tmp.txt | grep ";"` pode-se conferir os casos onde houve alguma adulteração, e com `diff tmp.txt CBO2002_LISTA.txt` pode-se conferir se as demais hipóteses de trabalho foram adequadas.

# Licença

[![](https://upload.wikimedia.org/wikipedia/commons/thumb/3/39/Cc-public_domain_mark_white.svg/64px-Cc-public_domain_mark_white.svg.png)](https://creativecommons.org/publicdomain/zero/1.0/deed.pt_BR) No Brasil [o *texto de Lei* (incluindo anexos e figuras) e seus complementos, são dados de *domínio público*](https://github.com/ppKrauss/licenses/blob/master/reports/implied-lex-BR-v1.md).  

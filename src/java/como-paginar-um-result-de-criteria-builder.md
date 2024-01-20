```java
    public Page<AlterarPrecoProdutoPainelDto> listarProdutos(GrupoEmpresa grupoEmpresa, FiltroAlterarProdutoPainelDto filtros, Pageable pageable) {
        CriteriaBuilder criteriaBuilder = getSession().getCriteriaBuilder();
        CriteriaQuery<AlterarPrecoProdutoPainelDto> criteriaQuery = criteriaBuilder.createQuery(AlterarPrecoProdutoPainelDto.class);

        Root<ProdutoEmpresa> root = criteriaQuery.from(ProdutoEmpresa.class);
        Join<ProdutoEmpresa, Promocao> promocaoJoin = root.join(ProdutoEmpresa_.PROMOCOES);
        Join<Promocao, GrupoDesconto> grupoDescontoJoin = promocaoJoin.join(Promocao_.GRUPO_DESCONTO);
        Join<ProdutoEmpresa, Empresa> empresaJoin = root.join(ProdutoEmpresa_.EMPRESA);
        Join<Empresa, GrupoEmpresa> grupoEmpresaJoin = empresaJoin.join(Empresa_.GRUPO_EMPRESA);

        criteriaQuery.select(criteriaBuilder.construct(AlterarPrecoProdutoPainelDto.class,
                root.get(ProdutoEmpresa_.DESCRICAO),
                empresaJoin.get(Empresa_.NOME_FANTASIA),
                grupoDescontoJoin.get(GrupoDesconto_.NOME),
                promocaoJoin.get(Promocao_.MODALIDADE_VALOR),
                root.get(ProdutoEmpresa_.VALOR),
                promocaoJoin.get(Promocao_.VALOR_DESCONTO),
                promocaoJoin.get(Promocao_.VALOR_DIFERENCA),
                promocaoJoin.get(Promocao_.VALOR_PERCENTUAL)
        ));

        List<Predicate> predicates = new ArrayList<>(Arrays.asList(criteriaBuilder.equal(grupoEmpresaJoin.get(GrupoEmpresa_.ID), grupoEmpresa.getId()),
                empresaJoin.in(filtros.getEmpresaIdLista()),
                grupoDescontoJoin.in(filtros.getGrupoDescontoIdLista()),
                criteriaBuilder.equal(root.get(ProdutoEmpresa_.STATUS), EnumStatus.ATIVO),
                criteriaBuilder.equal(root.get(ProdutoEmpresa_.COMBUSTIVEL), Boolean.FALSE)));

        criteriaQuery.where(predicates.toArray(new Predicate[0]));
        TypedQuery<AlterarPrecoProdutoPainelDto> typedQuery = getSession().createQuery(criteriaQuery);
        
        long totalElementos = typedQuery.getResultList().size();
        
        typedQuery.setFirstResult((int) pageable.getOffset());
        typedQuery.setMaxResults(pageable.getPageSize());
        List<AlterarPrecoProdutoPainelDto> dtoList = typedQuery.getResultList();
        
        return new PageImpl<>(dtoList, pageable, totalElementos);
    }

```

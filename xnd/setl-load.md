
##### LoadInterceptor流程

(1).LoadInterceptor.prepare()
    (2).interceptor.before()

    (3).dbDialect.getTransactionTemplate().execute(new TransactionCallback() {

                                       public Object doInTransaction(TransactionStatus status) {

                                            //添加同步标记
                                           (4).interceptor.transactionBegin(context, splitDatas, dbDialect);

                                             JdbcTemplate template = dbDialect.getJdbcTemplate();
                                             int[] affects = template.batchUpdate(sql, new BatchPreparedStatementSetter() {

                                                 public void setValues(PreparedStatement ps, int idx) throws SQLException {
                                                     doPreparedStatement(ps, dbDialect, lobCreator, splitDatas.get(idx));
                                                 }

                                                 public int getBatchSize() {
                                                     return splitDatas.size();
                                                 }
                                             });

                                           //添加同步标记
                                           (5).interceptor.transactionEnd(context, splitDatas, dbDialect);
                                       }
    }
    (6).interceptor.after()

(7).LoadInterceptor.commit()

(8).LoadInterceptor.error()，有错误的时候执行




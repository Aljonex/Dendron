---
id: zk1cvjc42lb0p7jpionq896
title: ATROPOS
desc: ''
updated: 1736418362385
created: 1736418339804
---
ATROPOS is the credit line facility
### Basic script
sqlStr = """ 
	EXPLAIN (<SQLQUERY>)
	""";
dao = context.getBean('agreementDAO'); 
rows = dao.sessionFactory.getCurrentSession().createSQLQuery(sqlStr).list(); 
output = ""; 
for(String row: rows){ 
	output += row; logger.info(row); 
} 
return output;
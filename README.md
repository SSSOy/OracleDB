
```SQL
-- 30, 40, 90번 부서원의 부서번호, 이름, 급여, GRADE를 조회하여 부서번호 순으로 정렬
-- GRADE : 급여가 3000보다 작으면 'A', 3000이상 7500미만이면 'B', 7500이상이면 'C'로 표시

SELECT DEPARTMENT_ID, FIRST_NAME, SALARY,
CASE 
WHEN SALARY < 3000 THEN 'A'
WHEN SALARY < 7500 THEN 'B'
ELSE 'C' 
END AS "GRADE" FROM EMPLOYEES WHERE DEPARTMENT_ID IN (30, 40, 90) ORDER BY 1;

SELECT DEPARTMENT_ID, FIRST_NAME, SALARY,
DECODE(TRUNC(SALARY / 3000), 0, 'A', DECODE(TRUNC(SALARY / 7500), 0, 'B', 'C')) AS "GRADE" 
FROM EMPLOYEES WHERE DEPARTMENT_ID IN (30, 40, 90) ORDER BY 1;


--부서별 최소급여가 80번 부서의 최소급여보다 많은 부서 조회
SELECT DEPARTMENT_ID, MIN(SALARY)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING MIN(SALARY)  < (SELECT MIN(SALARY) FROM EMPLOYEES WHERE DEPARTMENT_ID = 80)
ORDER BY 1;


--다중행 서브쿼리

SELECT MAX(SALARY) FROM EMPLOYEES GROUP BY DEPARTMENT_ID;

SELECT EMPLOYEE_ID, FIRST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES
WHERE SALARY IN (SELECT MAX(SALARY) FROM EMPLOYEES GROUP BY DEPARTMENT_ID)
ORDER BY DEPARTMENT_ID;


--업무가 SALES인 최소 한 명 이상의 사원보다 급여를 많이 받는 사원의 이름, 급여, 업무코드를 급여 순으로 조회
SELECT FIRST_NAME, SALARY, JOB_ID
FROM EMPLOYEES
WHERE SALARY > ANY (SELECT SALARY FROM EMPLOYEES WHERE JOB_ID LIKE 'SA%') AND JOB_ID NOT LIKE 'SA%'
ORDER BY 2;

SELECT FIRST_NAME, SALARY, JOB_ID
FROM EMPLOYEES
WHERE SALARY > ALL (SELECT SALARY FROM EMPLOYEES WHERE JOB_ID LIKE 'SA%') AND JOB_ID NOT LIKE 'SA%'
ORDER BY 2;


--Diana, Adam과 관리자 및 부서가 같은 사원의 이름, 관리자번호, 부서번호 조회
SELECT FIRST_NAME, MANAGER_ID, DEPARTMENT_ID
FROM EMPLOYEES
WHERE FIRST_NAME NOT IN('Diana', 'Adam')
AND (MANAGER_ID, DEPARTMENT_ID) IN (SELECT MANAGER_ID, DEPARTMENT_ID FROM EMPLOYEES WHERE FIRST_NAME IN ('Diana', 'Adam'));
SELECT MANAGER_ID, DEPARTMENT_ID FROM EMPLOYEES WHERE FIRST_NAME IN ('Diana', 'Adam');


--각 부서별로 최대 급여를 받는 사원과 최소급여를 받는 사원의 부서번호, 사원번호, 사원이름, 급여 조회
SELECT DEPARTMENT_ID, EMPLOYEE_ID, FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE (DEPARTMENT_ID, SALARY) IN (SELECT DEPARTMENT_ID, MAX(SALARY) FROM EMPLOYEES GROUP BY DEPARTMENT_ID) OR 
(DEPARTMENT_ID, SALARY) IN (SELECT DEPARTMENT_ID, MIN(SALARY) FROM EMPLOYEES GROUP BY DEPARTMENT_ID)
ORDER BY 1;

SELECT DEPARTMENT_ID, EMPLOYEE_ID, FIRST_NAME, SALARY
FROM EMPLOYEES SE
WHERE SALARY = (SELECT MAX(SALARY) FROM EMPLOYEES WHERE DEPARTMENT_ID = SE.DEPARTMENT_ID)
OR SALARY = (SELECT MIN(SALARY) FROM EMPLOYEES WHERE DEPARTMENT_ID = SE.DEPARTMENT_ID)
ORDER BY 1;



-- 사용자 정의함수 (STORED FUNCTION)

CREATE OR REPLACE FUNCTION FNSUM(N IN NUMBER)
RETURN NUMBER
IS
    TOT NUMBER := 0;
BEGIN
    FOR I IN 1 .. N LOOP
        TOT := TOT + I;
    END LOOP;
    RETURN TOT;
END;
/

SELECT FNSUM(5) FROM DUAL;


--주민번호로 성별 계산
CREATE OR REPLACE FUNCTION FNGENDER(S VARCHAR)
RETURN VARCHAR
IS
    RESULT VARCHAR(10) := '';
BEGIN
    IF LENGTH(S) != 14 THEN RAISE_APPLICATION_ERROR(-20001, '주민번호는 13자리입니다.');
    END IF;
    IF(SUBSTR(S, 8, 1) IN ('1', '3')) THEN
        RESULT := '남자';
    ELSE 
        RESULT := '여자';
    END IF;
    RETURN RESULT;
END;
/

SELECT FNGENDER('020627-4123456') AS "성별" FROM DUAL;
SELECT FNGENDER('920506-1234567') AS "성별" FROM DUAL;


--주민번호로 생일 계산
CREATE OR REPLACE FUNCTION FNBIRTH(S VARCHAR)
RETURN DATE
IS
    RESULT DATE;
BEGIN
    IF LENGTH(S) != 14 THEN RAISE_APPLICATION_ERROR(-20001, '주민번호는 13자리입니다.');
    END IF;
    RESULT := TO_DATE(SUBSTR(S, 0, 6));
    RETURN RESULT;
END;
/

SELECT FNBIRTH('020627-4123456') AS 생일 FROM DUAL;
SELECT FNBIRTH('920506-1234567') AS 생일 FROM DUAL;



--나이 계산(만 나이)
CREATE OR REPLACE FUNCTION FNAGE(S VARCHAR)
RETURN NUMBER
IS
    RESULT NUMBER;
BEGIN
    IF LENGTH(S) != 13 THEN RAISE_APPLICATION_ERROR(-20001, '주민번호는 13자리입니다.');
    END IF;
    RESULT := TRUNC(MONTHS_BETWEEN(SYSDATE, TO_DATE(SUBSTR(S, 0, 6))) / 12);
    RETURN RESULT;
END;
/

--한국 나이
CREATE OR REPLACE FUNCTION FNAGE(S VARCHAR)
RETURN NUMBER
IS
    RESULT NUMBER;
BEGIN
    IF LENGTH(S) != 13 THEN RAISE_APPLICATION_ERROR(-20001, '주민번호는 13자리입니다.');
    END IF;
    RESULT := TO_CHAR(SYSDATE, 'YYYY') - TO_CHAR(TO_DATE(SUBSTR(S, 0, 6)), 'YYYY') + 1;
    RETURN RESULT;
END;
/

SELECT FNAGE('0210104123456') AS 나이 FROM DUAL;
SELECT FNAGE('9205061234567') AS 나이 FROM DUAL;



```


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

```

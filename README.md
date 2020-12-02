We Can See Complete Microservice Architecture.we can create couple of services,registry, api gateway and hystrix and loadbalancing
---------------------------------------------------
create depart-service
---------------------
step1:create spring boot starter project

-web
-devtools
-h2
-jpa
-lombok

step2:create department model class 
package com.chandra.departmentservice.entity;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long departmentId;
    private String departmentName;
    private String departmentAddress;
    private String departmentCode;
}
step3:create a department repository interface
package com.chandra.departmentservice.repository;


import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.chandra.departmentservice.entity.Department;

@Repository
public interface DepartmentRepository extends JpaRepository<Department, Long> {

    Department findByDepartmentId(Long departmentId);
}
step4:create a service class DepartmentService
package com.chandra.departmentservice.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.chandra.departmentservice.entity.Department;
import com.chandra.departmentservice.repository.DepartmentRepository;

@Service
@Slf4j
public class DepartmentService {

    @Autowired
    private DepartmentRepository departmentRepository;

    public Department saveDepartment(Department department) {
        log.info("Inside saveDepartment of DepartmentService");
        return departmentRepository.save(department);
    }

    public Department findDepartmentById(Long departmentId) {
        log.info("Inside saveDepartment of DepartmentService");
        return departmentRepository.findByDepartmentId(departmentId);
    }
}

step5:create a department controller

package com.chandra.departmentservice.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.chandra.departmentservice.entity.Department;
import com.chandra.departmentservice.service.DepartmentService;

import lombok.extern.slf4j.Slf4j;

@RestController
@RequestMapping("/departments")
@Slf4j
public class DepartmentController {

    @Autowired
    private DepartmentService departmentService;

    @PostMapping("/")
    public Department saveDepartment(@RequestBody Department department) {
        log.info("Inside saveDepartment method of DepartmentController");
        return  departmentService.saveDepartment(department);
    }

    @GetMapping("/{id}")
    public Department findDepartmentById(@PathVariable("id") Long departmentId) {
        log.info("Inside findDepartmentById method of DepartmentController");
        return departmentService.findDepartmentById(departmentId);
    }

}


step6:create application.yml file

server:
 port: 9001

step7: run the application

step8:open the postman client and give the following url

POST   http://localhost:9001/departments/

{
	"departmentName":"IT",
	"departmentAddress":"West Venkatapuram",
	"departmentCode":"IT 006"
}

Employee-Service Application Development steps
-----------------------------------------------
step1:create a spring boot starter project same as above.

step2:create a enity class Employee
package com.chandra.employeeservice.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Employee {
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	private Long employeeId;
	private String firstName;
	private String lastName;
	private String email;
	private Long departmentId;

}

step3:create a repository interface

package com.chandra.employeeservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.chandra.employeeservice.entity.Employee;
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

	 Employee findByEmployeeId(Long employeeId);
}

step4:create a service class
package com.chandra.employeeservice.services;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import com.chandra.employeeservice.VO.Department;
import com.chandra.employeeservice.VO.ResponseTemplateVO;
import com.chandra.employeeservice.entity.Employee;
import com.chandra.employeeservice.repository.EmployeeRepository;

import lombok.extern.slf4j.Slf4j;

@Service
@Slf4j
public class EmployeeService {

	@Autowired
	private EmployeeRepository employeeRepository;

	@Autowired
	private RestTemplate restTemplate;

	public Employee saveEmployee(Employee employee) {
		log.info("Inside saveEmployee of EmployeeService ");
		return employeeRepository.save(employee);
	}

	public ResponseTemplateVO getUserWithDepartment(Long employeeId) {
		log.info("Inside getUserWithDepartment of UserService");
		ResponseTemplateVO vo = new ResponseTemplateVO();
		Employee employee = employeeRepository.findByEmployeeId(employeeId);

		Department department = restTemplate
				.getForObject("http://localhost:9001/departments/" + employee.getDepartmentId(), Department.class);

		vo.setEmployee(employee);
		vo.setDepartment(department);

		return vo;
	}
}

step5:create a employee controller class

package com.chandra.employeeservice.controller;


import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import com.chandra.employeeservice.VO.ResponseTemplateVO;
import com.chandra.employeeservice.entity.Employee;
import com.chandra.employeeservice.services.EmployeeService;

@RestController
@RequestMapping("/users")
@Slf4j
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    @PostMapping("/")
    public Employee saveEmployee(@RequestBody Employee employee) {
        log.info("Inside saveUser of UserController");
        return employeeService.saveEmployee(employee);
    }

    @GetMapping("/{id}")
    public ResponseTemplateVO getUserWithDepartment(@PathVariable("id") Long employeeId) {
        log.info("Inside getUserWithDepartment of UserController");
        return employeeService.getUserWithDepartment(employeeId);
    }


}

step6:create value object (Department,ResponseTemplateVO(Department,Employee)

package com.chandra.employeeservice.VO;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Department {

    private Long departmentId;
    private String departmentName;
    private String departmentAddress;
    private String departmentCode;
}


------
package com.chandra.employeeservice.VO;

import com.chandra.employeeservice.entity.Employee;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class ResponseTemplateVO {

    private Employee employee;
    private Department department;
}


step7:To communicate one application to another application crate RestTemplate in main configuration class
package com.chandra.employeeservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication

public class EmployeeServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(EmployeeServiceApplication.class, args);
	}
	
	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}

}

step8:use the following url in post man client and insert employee details

POST : http://localhost:9002/employees/

{
	"employeeId":1,
	"firstName":"Sai",
	"lastName":"Charan",
	"email":"saicharan@gmail.com",
	"departmentId":1
}

step9: to get employee id based department details also use postman client 

GET :http://localhost:9002/employees/1

{
    "employee": {
        "employeeId": 1,
        "firstName": "Sai",
        "lastName": "Charan",
        "email": "saicharan@gmail.com",
        "departmentId": 1
    },
    "department": {
        "departmentId": 1,
        "departmentName": "IT",
        "departmentAddress": "West Venkatapuram",
        "departmentCode": "IT 006"
    }
}














# Proyecto-MySQL-ll
# Multilevel Digital Commerce Platform – Relational Database System

## 📘 Overview

This project implements a **relational database system in MySQL** to support the operation of a **digital platform** designed for the **multilevel commercialization of products and services**. The system enables comprehensive management of companies, customers, products, reviews, memberships, benefits, and geographic locations, following a **scalable and modular approach**.

## 🔧 Technical Justification

The increasing demand for B2B and B2C platforms that support **personalization**, **quality assessment**, **user segmentation**, and **loyalty programs** highlights the need for robust, normalized, and efficient database solutions. This system follows best practices in relational modeling, referential integrity, and future scalability.

## 🎯 System Objectives

Develop and implement a **normalized MySQL database** to efficiently manage:

- Clients and registered companies  
- Product and service catalogs  
- Geographic segmentation  
- Favorites and preferences  
- Product evaluations via surveys  
- Membership plans and associated benefits  
- Audience-based quality metrics

## 🧩 Database Model & Structure

### 🌍 Geographic Structure

- `countries` – Countries  
- `stateregions` – States or departments  
- `citiesormunicipalities` – Cities or municipalities  

Used for precise user and company geolocation.

### 🏢 Entity Management

- `companies` – Stores company data (location, category, audience, etc.)  
- `customers` – Stores customer profiles, preferences, and interaction history  

### 📦 Product Catalog

- `products` – Product data (description, price, image, category)  
- `companyproducts` – Many-to-many link between companies and products with price & unit customizations  

### ⭐ Evaluations & Metrics

- `polls` – Configurable surveys for product and company evaluations  
- `rates` – Customer ratings of specific products by company  
- `quality_products` – Advanced satisfaction metrics linked to polls  

### 👤 User Personalization

- `favorites` and `details_favorites` – User-managed product interest lists  
- `audiences` – Demographic or preference-based segmentation  

### 💎 Membership & Benefits System

- `memberships` – Commercial membership plans  
- `membershipperiods` – Duration and cost of each plan  
- `benefits` – Privileges tied to memberships  
- `audiencebenefits` & `membershipbenefits` – Access control by audience or membership type  

## 📐 Normalization & Integrity

- Designed up to **Third Normal Form (3NF)**  
- All relationships enforced via **foreign keys** (MySQL InnoDB engine)  
- Redundancy reduction, semantic integrity, and optimized queries guaranteed  

## 💻 Implementation Details

- **DBMS:** MySQL 8.x  
- **Storage Engine:** InnoDB  
- **Recommended Tools:** MySQL Workbench, DBeaver  
- **Query Language:** Standard SQL + MySQL-specific extensions  

## 📈 Scalability & Security

- Supports horizontal scaling (new categories, products, regions, memberships)  
- Role-based access model to be implemented at application level  
- Enforced data integrity through constraints (UNIQUE, NOT NULL, field lengths)  

## 🚀 Use Cases

- API back-end for mobile/web platforms  
- Admin dashboards  
- Customer/product analytics  
- Membership-based benefits delivery  

## 🧑‍💻 Author

Developed by [Your Name or Team Name]  
For academic, research, or commercial purposes

## 📄 License

[Choose a license – e.g., MIT, GPLv3, etc.]

---

**Note:** This database serves as a foundational layer for further development into distributed systems or microservices architecture.

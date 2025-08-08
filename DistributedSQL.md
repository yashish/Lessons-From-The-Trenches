# 🌐 The Need for Distributed SQL in the Industry

Industries must differentiate and innovate to stay competitive. A canonical example is the **finance industry**, which continues to **lag behind** others in adopting modern data infrastructure.

---

## 🏦 Legacy Challenges in Financial Services

> *From YugabyteDB:*

Despite the rise of **cloud infrastructures** and **cloud-native applications**, many financial services firms still rely on legacy RDBMSs like:

- Oracle  
- SQL Server  
- Db2  

These systems support business-critical applications, but they were **not designed** for the modern world of:

- Cloud computing  
- Geo-distributed applications  

These monolithic databases were built **long before** cloud architecture was even imagined.

---

## ⚠️ Limitations of Monolithic Databases

Monolithic databases attempt to meet growing demands by **adding capacity**, but:

- Scaling up is **slow and complex**  
- Scaling down is **equally difficult**  
- High availability requires a **standby machine** with **replication**

However, replication is often **asynchronous**, which introduces:

- Performance compromises  
- Additional complications  

This architecture **fails** when trying to support **geo-distributed microservices** that require **millisecond-level response times**.

---

## ✅ The Solution: Distributed SQL

To meet modern demands, the database must be:

- **Agile**  
- **Horizontally scalable**  
- **Continuously available**  
- **Built for the cloud** (not retrofitted)

> The database needs to be **Distributed SQL**.

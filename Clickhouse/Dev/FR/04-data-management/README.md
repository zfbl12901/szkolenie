# 4. Gestion des Données

## 🎯 Objectifs

- Créer des tables depuis Java
- Gérer les schémas
- Importer/exporter des données
- Gérer les partitions

## Création de tables

### Créer une table

```java
String createTableSQL = """
    CREATE TABLE IF NOT EXISTS events (
        id UInt64,
        event_date Date,
        event_time DateTime,
        user_id UInt32,
        event_type String,
        value Float64
    )
    ENGINE = MergeTree()
    PARTITION BY toYYYYMM(event_date)
    ORDER BY (event_date, user_id)
    """;

try (Connection conn = ConnectionManager.getConnection();
     Statement stmt = conn.createStatement()) {
    stmt.execute(createTableSQL);
    System.out.println("Table created successfully!");
}
```

## Gestion des schémas

### Vérifier l'existence d'une table

```java
public boolean tableExists(String tableName) throws SQLException {
    String sql = "EXISTS TABLE " + tableName;
    
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        return rs.next() && rs.getInt(1) == 1;
    }
}
```

### Obtenir la structure d'une table

```java
public void describeTable(String tableName) throws SQLException {
    String sql = "DESCRIBE TABLE " + tableName;
    
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        
        System.out.println("Column\tType");
        while (rs.next()) {
            System.out.println(rs.getString("name") + "\t" + 
                             rs.getString("type"));
        }
    }
}
```

## Import de données

### Depuis CSV

```java
String insertSQL = "INSERT INTO events FROM INFILE '/path/to/file.csv' FORMAT CSV";

try (Connection conn = ConnectionManager.getConnection();
     Statement stmt = conn.createStatement()) {
    stmt.execute(insertSQL);
}
```

### Depuis fichier local

```java
public void importFromFile(String filePath) throws SQLException, IOException {
    String sql = "INSERT INTO events FORMAT CSV";
    
    try (Connection conn = ConnectionManager.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql);
         BufferedReader reader = Files.newBufferedReader(Paths.get(filePath))) {
        
        // Lire et insérer ligne par ligne
        String line;
        while ((line = reader.readLine()) != null) {
            // Parser et insérer
        }
    }
}
```

## Export de données

### Vers CSV

```java
public void exportToCSV(String outputPath) throws SQLException, IOException {
    String sql = "SELECT * FROM events";
    
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql);
         BufferedWriter writer = Files.newBufferedWriter(Paths.get(outputPath))) {
        
        // Écrire l'en-tête
        ResultSetMetaData metaData = rs.getMetaData();
        for (int i = 1; i <= metaData.getColumnCount(); i++) {
            writer.write(metaData.getColumnName(i));
            if (i < metaData.getColumnCount()) writer.write(",");
        }
        writer.newLine();
        
        // Écrire les données
        while (rs.next()) {
            for (int i = 1; i <= metaData.getColumnCount(); i++) {
                writer.write(rs.getString(i));
                if (i < metaData.getColumnCount()) writer.write(",");
            }
            writer.newLine();
        }
    }
}
```

## Gestion des partitions

### Lister les partitions

```java
public void listPartitions(String tableName) throws SQLException {
    String sql = "SELECT partition, rows, bytes_on_disk " +
                 "FROM system.parts " +
                 "WHERE table = ? AND active = 1";
    
    try (Connection conn = ConnectionManager.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        pstmt.setString(1, tableName);
        
        try (ResultSet rs = pstmt.executeQuery()) {
            while (rs.next()) {
                System.out.println("Partition: " + rs.getString("partition") +
                                 ", Rows: " + rs.getLong("rows"));
            }
        }
    }
}
```

### Supprimer une partition

```java
public void dropPartition(String tableName, String partition) throws SQLException {
    String sql = "ALTER TABLE " + tableName + " DROP PARTITION ?";
    
    try (Connection conn = ConnectionManager.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        pstmt.setString(1, partition);
        pstmt.executeUpdate();
    }
}
```

---

**Prochaine étape :** [Performance et Optimisation](./05-performance/README.md)


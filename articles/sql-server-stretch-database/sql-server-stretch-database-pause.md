<properties
	pageTitle="Pausar e retomar a migração de dados (Stretch Database) | Microsoft Azure"
	description="Saiba como pausar ou retomar a migração de dados no Azure."
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/14/2016"
	ms.author="douglasl"/>

# Pausar e retomar a migração de dados (Stretch Database)

Para pausar ou retomar a migração de dados no Azure, selecione **Stretch** de uma tabela no SQL Server Management Studio e selecione **Pausar** para pausar a migração de dados ou **Retomar** para retomá-la. Também é possível usar o Transact-SQL para pausar ou retomar a migração de dados.

Pause a migração de dados em tabelas individuais quando desejar solucionar problemas no servidor local ou para maximizar a largura de banda de rede disponível.

## Pausar migração de dados

### Usar o SQL Server Management Studio para pausar a migração de dados

1.  No SQL Server Management Studio, no Pesquisador de Objetos, selecione a tabela habilitada para Stretch da qual deseja pausar a migração de dados.

2.  Clique com o botão direito do mouse e selecione **Stretch** e, em seguida, selecione **Pausar**.

### Usar o Transact-SQL para pausar a migração de dados
Execute o comando a seguir.

```tsql
USE <Stretch-enabled database name>;
GO
ALTER TABLE <Stretch-enabled table name>  
    SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = PAUSED ) ) ;  
GO
```

## Retomar a migração de dados

### Usar o SQL Server Management Studio para retomar a migração de dados

1.  No SQL Server Management Studio, no Pesquisador de Objetos, selecione a tabela habilitada para Stretch da qual deseja retomar a migração de dados.

2.  Clique com o botão direito do mouse e selecione **Stretch** e, em seguida, selecione **Retomar**.

### Usar o Transact-SQL para retomar a migração de dados
Execute o comando a seguir.

```tsql
USE <Stretch-enabled database name>;
GO
ALTER TABLE <Stretch-enabled table name>   
    SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = OUTBOUND ) ) ;  
 GO
```

## Verifique se a migração está ativa ou em pausa

### Use o SQL Server Management Studio para verificar se a migração está ativa ou em pausa
No SQL Server Management Studio, abra o **Stretch Database Monitor** e verifique o valor da coluna **Estado de migração**. Para obter mais informações, confira [Monitor and troubleshoot data migration](sql-server-stretch-database-monitor.md) (Monitorar e solucionar problemas de migração de dados).

### Use o Transact-SQL para verificar se a migração está ativa ou em pausa
Consulte a exibição de catálogo **sys.remote\_data\_archive\_tables** e verifique o valor da coluna **is\_migration\_paused**. Para obter mais informações, confira [sys.remote\_data\_archive\_tables](https://msdn.microsoft.com/library/dn935003.aspx).

## Consulte também

[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx) [Monitorar e solucionar problemas de migração de dados](sql-server-stretch-database-monitor.md)

<!---HONumber=AcomDC_0622_2016-->
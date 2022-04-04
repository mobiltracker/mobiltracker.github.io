# How to Safely Migrate a SQL Table

## Introduction

This documentation will show how to safely migrate a SQL Table. As an example we'll use the migration of the Routes table to RoutesTemplates.

## Step by Step

#### Setup inicial

- [ ] Crie uma issue no repositório do mobiltracker-data para a migração da tabela

  - [ ] Copie esse tutorial no corpo da issue
  - [ ] Atualize o checklist conforme for progredindo com a migração

- [ ] Crie a nova tabela

#### Setup de triggers

- [ ] Crie triggers que mantenham a nova tabela atualizada com a tabela atual

  - [ ] Criar trigger que deleta itens da tabela nova

    > É importante que o trigger de DELETE venha antes do INSERT para evitar o seguinte cenário:
    >
    > - INSERT da linha X na tabela antiga e replicada para a tabela nova por trigger
    > - DELETE da linha X na tabela antiga
    > - **PROBLEMA**: agora a tabela nova tem uma linha que deveria ter sido deletada

    ```sql
    -- Use essa query apenas como referência
    CREATE TRIGGER RoutesToRouteTemplatesReplicationOnDeleteTrigger ON [dbo].[Routes]
    FOR DELETE
    AS
        -- Remove deleted lines from the new table
        DELETE FROM Route_RouteTemplates
        WHERE RouteTemplateId in (SELECT RouteId FROM deleted)
    GO
    ```

  - [ ] Criar trigger que atualiza itens da tabela nova

    > É importante que o trigger de UPDATE venha antes do INSERT para evitar o seguinte cenário:
    >
    > - INSERT da linha X na tabela antiga e replicada para a tabela nova por trigger
    > - UPDATE da linha X na tabela antiga
    > - **PROBLEMA**: agora a tabela nova tem uma linha com dados desatualizados

    ```sql
    -- Use essa query apenas como referência
    CREATE TRIGGER RoutesToRouteTemplatesReplicationOnUpdateTrigger ON [dbo].[Routes]
    FOR UPDATE
    AS
        BEGIN TRANSACTION;

        SET NOCOUNT ON;

        DECLARE @RouteId bigint,
            @Name NVARCHAR(50),
            @VanId bigint,
            @SchedulerTime datetime,
            @Mon bit,
            @Tue bit,
            @Wed bit,
            @Thu bit,
            @Fri bit,
            @Sat bit,
            @Sun bit;

        DECLARE insertedCursor CURSOR FOR
        SELECT routeid, Name, VanId, SchedulerTime, Mon, Tue, Wed, Thu, Fri, Sat, Sun
        FROM inserted;

        OPEN insertedCursor

        FETCH NEXT FROM insertedCursor INTO @RouteId, @Name, @VanId, @SchedulerTime, @Mon, @Tue, @Wed, @Thu, @Fri, @Sat, @Sun

        -- Iterate all updated lines
        WHILE @@FETCH_STATUS = 0
        BEGIN

            UPDATE Route_RouteTemplates
            set
                Name = @Name,
                VanId = @VanId,
                SchedulerTime = @SchedulerTime,
                Mon = @Mon,
                Tue = @Tue,
                Wed = @Wed,
                Thu = @Thu,
                Fri = @Fri,
                Sat = @Sat,
                Sun = @Sun
            where RouteTemplateId = @RouteId

            -- Get next element
            FETCH NEXT FROM insertedCursor INTO @RouteId, @Name, @VanId, @SchedulerTime, @Mon, @Tue, @Wed, @Thu, @Fri, @Sat, @Sun
        END
        CLOSE insertedCursor;
        DEALLOCATE insertedCursor;

        COMMIT TRANSACTION;
    GO
    ```

  - [ ] Crie uma trigger na tabela antiga que replique os novos inserts para a tabela nova

    ```sql
    -- Use essa query apenas como referência
    CREATE TRIGGER RoutesToRouteTemplatesReplicationOnUpdateTrigger ON [dbo].[Routes]
    FOR UPDATE
    AS
        BEGIN TRANSACTION;

        SET NOCOUNT ON;

        DECLARE @RouteId bigint,
            @Name NVARCHAR(50),
            @VanId bigint,
            @SchedulerTime datetime,
            @Mon bit,
            @Tue bit,
            @Wed bit,
            @Thu bit,
            @Fri bit,
            @Sat bit,
            @Sun bit;

        DECLARE insertedCursor CURSOR FOR
        SELECT routeid, Name, VanId, SchedulerTime, Mon, Tue, Wed, Thu, Fri, Sat, Sun
        FROM inserted;

        OPEN insertedCursor

        FETCH NEXT FROM insertedCursor INTO @RouteId, @Name, @VanId, @SchedulerTime, @Mon, @Tue, @Wed, @Thu, @Fri, @Sat, @Sun

        -- Iterate all updated lines
        WHILE @@FETCH_STATUS = 0
        BEGIN

            UPDATE Route_RouteTemplates
            set
                Name = @Name,
                VanId = @VanId,
                SchedulerTime = @SchedulerTime,
                Mon = @Mon,
                Tue = @Tue,
                Wed = @Wed,
                Thu = @Thu,
                Fri = @Fri,
                Sat = @Sat,
                Sun = @Sun
            where RouteTemplateId = @RouteId

            -- Get next element
            FETCH NEXT FROM insertedCursor INTO @RouteId, @Name, @VanId, @SchedulerTime, @Mon, @Tue, @Wed, @Thu, @Fri, @Sat, @Sun
        END
        CLOSE insertedCursor;
        DEALLOCATE insertedCursor;

        COMMIT TRANSACTION;
    GO
    ```

#### Migração de dados

- [ ] Migre os dados restantes da tabela antiga para a tabela nova

  > A partir daqui, a tabela nova estará sempre atualizada com a tabela antiga.

  ```sql
  -- Use essa query apenas como referência
  set IDENTITY_INSERT Route_RouteTemplates on;

  INSERT INTO Route_RouteTemplates
      (RouteTemplateId, [Name], VanId, SchedulerTime, mon, tue, wed, thu, fri, sat, sun)
  SELECT r.RouteId, r.Name, r.VanId, r.SchedulerTime, r.Mon, r.Tue, r.Wed, r.Thu, r.Fri, r.Sat, r.Sun
  FROM Routes as R
  LEFT JOIN Route_RouteTemplates
      as rt on r.RouteId = rt.RouteTemplateId

  -- filter columns that already are on the new table
  where rt.RouteTemplateId is null;

  set IDENTITY_INSERT Route_RouteTemplates off;
  ```

- [ ] Migre todas as Foreign Keys que apontam para a tabela antiga para a tabela nova

#### Migração de leitura

- [ ] Migrar as leituras da tabela antiga para a tabela nova

  > É MUITO IMPORTANTE que as migrações de leitura aconteçam primeiro, para evitar o seguinte cenário:
  >
  > - UPDATE da linha X na tabela nova
  > - SELECT da linha X na tabela antiga
  > - **PROBLEMA**: O SELECT retornará um dado desatualizado pois o UPDATE aconteceu apenas na tabela nova.

  - [ ] Scripts
  - [ ] Processadores
  - [ ] Usos na API

    > o Entity Framework não permite que a leitura de uma classe seja feito de uma tabela e escrita em outra, por isso a estratégia será criar uma classe nova e substituir a antiga classe por ela

    - [ ] Crie uma nova classe a partir da classe antiga
    - [ ] Mapeie no DbContext para a tabela nova
    - [ ] Marque a classe antiga como deprecada
    - [ ] Substitua os usos de leitura pela classe nova
    - [ ] Confirme que todos os testes ainda estão passando

- [ ] Confirme que a tabela não está sendo acessada de mais nenhum outro lugar

  - [ ] Confirme com o resto da equipe se ninguém mais sabe de algum outro lugar onde a tabela é acessada
  - [ ] Acompanhe o uso da tabela
    ```sql
    -- Utilize essa query apenas como referência
    select
        last_user_scan as last_select,
        last_user_update as last_update_or_insert
    from sys.dm_db_index_usage_stats
    where object_id = object_id('dbo.Routes')
    ```
  - [ ] Cheque periodicamente o uso da tabela ao longo do tempo desejado (recomendo por um período entre 1 dia e 1 mês, 1 semana sendo um bom meio-termo)

#### Migração de DELETE e UPDATE

- [ ] Migrar as operações de delete e update da tabela antiga para a tabela nova

> É IMPORTANTE que as operações de delete e update da tabela antiga sejam migradas antes da operação de insert para evitar o seguinte cenário:
>
> - INSERT da linha X na tabela NOVA [pois já foi migrado]
> - DELETE da linha X na tabela ANTIGA [pois ainda não foi migrado]
> - **PROBLEMA**: Erro, pois a linha X não existe na tabela antiga pois foi criada direta na tabela nova

- [ ] Scripts
- [ ] Processadores
- [ ] Usos na API

  - [ ] Substitua os usos de exclusão e atualização da classe antiga pela classe nova
  - [ ] Confirme que todos os testes ainda estão passando

#### Migração de INSERT

- [ ] Migrar as operações de insert da tabela antiga para a tabela nova

  - [ ] Scripts
  - [ ] Processadores
  - [ ] Usos na API
    - [ ] Substitua todos os usos da classe antiga pela classe nova (devem ter sobrado apenas operações de insert)
    - [ ] Delete a classe antiga
    - [ ] Confirme que todos os testes ainda estão passando

#### Acompanhamento final

- [ ] Acompanhe o uso da tabela periodicamente para confirmar que ela deixou de ser usada

- [ ] Prefixe o nome da tabela com "\_\_TO_BE_DELETED\_\_"

- [ ] Após algumas semanas, delete a tabela antiga

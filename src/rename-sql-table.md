# How to rename a SQL Table

Most table migrations will affect a lot of different proccesses and can cause irreversible damages.
So, before starting any migration, make sure that the whole team is aware of the changes and make sure that they are worth it.

We have different migration strategies depending on the characteristics of the table.
Take the following quiz to find the one that best suit your needs.

Does the table have only one or very few updating processes?

> **NO**: Next question.

> **YES**: Use the View Strategy
>
> The idea behind this strategy is to replace the current table with a view of the new one and then quickly point the updating process to the new table. This change wont break the reading processes because they'll read from the view and so you can migrate them at your own pace.
>
> How to do it:
>
> - Create new table
> - [Quickly] Delete old table, create view pointing to the new table with the name of the old table and migrate the updating process to the new table
> - Migrate the read processes to the new table
> - Make sure the view is not being used anymore
> - Delete old table

Is the table huge?

> **YES**: Next question

> **NO**: Use the Duplication Strategy
>
> The idea behind this strategy is to duplicate the old table to the new one and keep it updated so that you can migrate it at your own pace.
> This strategy is ideal for tables that are accessed and updated by multiple processes, but it takes a lot of space due to the duplication, so it is not recommended for tables with too much data.
>
> We have a issue template with a step-by-step tutorial on the mobiltracker-data repository. Create a issue from this template to follow this tutorial and to bring visibility to this migration.
>
> Click [here](https://github.com/mobiltracker/mobiltracker-data/blob/master/.github/ISSUE_TEMPLATE/mwp--multiple-writing-processes--table-renaming-migration-.md) to see the template.

???

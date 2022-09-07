## parser/create
- struct ParsedColumn -> name, datatype, is_pk, is_nullable
- struct CreateQuery (new) -> table_name, columns

## parser/create
- struct InsertQuery (new) -> table_name, columns, values

## parser/select
- enum Binary
- enum Operator
- struct Expression
- struct SelectQuery (new, insert_projections) -> from, projection, where_expressions

## database
- struct Database (new, table_exists, get_table, get_table_mut) -> tables

## table
- enum DataType (new, fmt)
- enum ColumnHeader (new, get_mut_index)
- enum ColumnData (get_ser, get_ser_by_index, get_ser_by_scanning)
- enum ColumnIndex (get_idx_data, get_idx_data_by_range)
- struct Table (new, get_column, does_violate_unique_constraint, insert_row, select_data, execute_select_query_without_index, execute_select_query, print_table, print_table_data, column_exist)




```go

use parser::create::CreateQuery; insert::InsertQuery; select::SelectQuery;
use sqlparser::ast::Statement; dialect::MySqlDialect; parser::Parser;
mod database; parser; table;

fn process_command(query: String, db: &mut database::Database) {
	let dialect = MySqlDialect {};
	let statements = &Parser::parse_sql(&dialect, query).unwrap_or_default();

	for s in statements {
		// match s
		
		// Statement::CreateTable { .. }
		let cq = CreateQuery::new(s).unwrap();
		db.tables.push(table::Table::new(cq));

		// Statement::Insert { .. }
		let iq = InsertQuery::new(s);
		println!("cols = {:?}\n vals = {:?}", iq.columns, iq.values);
		if (!db.table_exists(iq.table_name.to_string()) { return; }
		let db_table = db.get_table_mut(iq.table_name.to_string());
		match iq.columns.iter().all(|c| db_table.column_exist(c.to_string())) {
		for value in &iq.values {
			match db_table.does_violate_unique_constraint(&iq.columns, value)
			db_table.insert_row(&iq.columns, &values);
		}
		
		// Statement::Query(_q) { .. }
		let mut sq = SelectQuery::new(&s);
		if (db.table_exists((&sq.from).to_string())) { return; }
		let db_table = db.get_table((&sq.from).to_string());
		let cloned_projection = sq.projection.clone();
		for p in &cloned_projection {
			if p == "*" { sq.insert_projections(db_table.columns.iter().map(|c| c.name.to_string()).collect::<Vec<String>>()); }
		}
		for col in &sq.projection {
				if !db_table.column_exist((&col).to_string()) { return; }
		}
		db_table.execute_select_query(&sq);
	}
}

fn main() {
    let mut command = String::new();
    let mut db = database::Database::new();
    loop {
			process_command(command.trim().to_string(), &mut db);
			handle_meta_command(cmd, &mut db);
    }
}
```


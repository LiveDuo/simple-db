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




```rust

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


```rust
enum ColumnIndex { Int(BTreeMap<i32, usize>), Str(BTreeMap<String, usize>), Bool(BTreeMap<bool, usize>), None, }
enum DataType { Int, Str, Float, Bool, Invalid, }
enum ColumnData { Int(Vec<i32>), Str(Vec<String>), Float(Vec<f32>), Bool(Vec<bool>), None, }

struct ColumnHeader { name: String, datatype: DataType, is_indexed: bool, index: ColumnIndex, is_primary_key: bool, }
struct Table { columns: Vec<ColumnHeader>, name: String, rows: HashMap<String, ColumnData>, }
struct Database { tables: Vec<Table>, }
```

## Database Schema

- tables __Vec<Table>__
	- rows __HashMap<String,ColumnData>__ `enum`
	- columns __Vec<ColumnHeader>__ `struct`
		name __String__
		datatype __DataType__ `enum` -> primitive
		is_indexed __bool__
		index __ColumnIndex__ `enum` -> BTreeMap
		is_primary_key __bool__


### Dry Run

- "create table _table_ ..."
```rust
process_command(query, &mut db);
	let cq = CreateQuery::new(s).unwrap();
		Ok(CreateQuery { table_name: table_name.to_string(), columns: parsed_columns, });
  db.tables.push(Table::new(cq));
```

- "insert * from _table_ ...;"
```rust
process_command(query, &mut db);
	let iq = InsertQuery::new(s).unwrap();
	if !db.table_exists(iq.table_name.to_string()) return;
	let db_table = db.get_table_mut(iq.table_name.to_string());
	if !iq.columns.iter().all(|c| db_table.column_exist(c.to_string())) return;
	for value in &iq.values {
		if !db_table.does_violate_unique_constraint(&iq.columns, value) return;
		db_table.insert_row(&iq.columns, &iq.values);
	}
```

- "select into _table_ ..."
```rust
process_command(query, &mut db);
	let select_query = SelectQuery::new(&s); // parses the query
		// performs more parsing
		Ok(SelectQuery { from: name, projection, where_expressions, });
	if !db.table_exists((&sq.from).to_string() return // checks if table exists
	let db_table = db.get_table((&sq.from).to_string()); // gets the table
	for p in &sq.projection.clone() { if p == "*" sq.insert_projections(new_projections); } // deals with "*" symbol
	for col in &sq.projection { if !db_table.column_exist((&col).to_string()) return; } // checks if columns exist
	db_table.execute_select_query(&sq); // executes the query
        let where_expr = sq.where_expressions.first().unwrap() // gets the first where expression
		let col = self.get_column(where_expr.left.to_string()); // gets the column name
		let row = self.rows.get(&col.name).unwrap(); // gets the 
		let indices = row.get_serialized_col_data_by_scanning(&where_expr);
		let projected_data = sq.projection.iter()
			.map(|col_name| self.rows.get(&col_name.to_string()).unwrap())
			.map(|col_data| col_data.get_serialized_col_data_by_index(&indices))
			.collect::<Vec<Vec<String>>>();
		return projected_data;

```


```rust
[Table { 
	columns: [
		ColumnHeader { name: "id", datatype: Int, is_indexed: true, index: Int({}), is_primary_key: true }, 
		ColumnHeader { name: "name", datatype: Str, is_indexed: false, index: Str({}), is_primary_key: false }, 
		ColumnHeader { name: "score", datatype: Float, is_indexed: false, index: None, is_primary_key: false }], 
	name: "users", 
	rows: {"score": Float([]), "id": Int([]), "name": Str([])} 
}]

[Table { 
	columns: [
		ColumnHeader { name: "id", datatype: Int, is_indexed: true, index: Int({1: 0, 2: 1}), is_primary_key: true }, 
		ColumnHeader { name: "name", datatype: Str, is_indexed: false, index: Str({"\"John Doe\"": 1}), is_primary_key: false }, 
		ColumnHeader { name: "score", datatype: Float, is_indexed: false, index: None, is_primary_key: false }
	], 
	name: "users", 
	rows: {"id": Int([1, 2]), "score": Float([57.23, 57.23]), "name": Str(["\"John Doe\"", "\"John Doe\""])} 
}]

```

✅ create table posts (id int PRIMARY KEY, title string, dec string);
✅ insert into posts(id, title, dec) values(1, "new", "dec");
```rust
db = [Table { 
	columns: [
		ColumnHeader { name: "id", datatype: Int, is_indexed: true, index: Int({1: 0, 2: 1}), is_primary_key: true }, 
		ColumnHeader { name: "title", datatype: Str, is_indexed: true, index: Str({"\"new\"": 0, "\"new2\"": 1}), is_primary_key: false }, 
		ColumnHeader { name: "dec", datatype: Str, is_indexed: true, index: Str({"\"dec\"": 0, "\"dec2\"": 1}), is_primary_key: false }], 
	name: "posts", 
	rows: {"id": Int([1, 2]), "title": Str(["new", "new2"]), "dec": Str(["dec", "dec2"])} 
}]
```


✅ select * from posts; / select * from posts where id = 1;
```js
let result = []
for (let (id, i) of db.rows.id ) {
	let row = [rows.id[i], rows.title[i], rows.dec[i]]
}	result.push(row)
return result;
```

```js
input: id = 1

let column = db.columns.find(c => c.name === 'id')
let idx = column.index[id] // 0
let row = [rows.id[idx], rows.title[idx], rows.dec[idx]]
return row

```
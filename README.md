Cairo is a programming language created by [Starkware](https://www.cairo-lang.org/), which can be used to develop scalable, layer two blockchain applications.
For complete and accurate information on how to use the language, see the [official documentation](https://www.cairo-lang.org/docs/).

This site is meant to provide some basic examples on some different functionalities of the Cairo language and some of the most common security concerns that are prevalent to this language and ecosystem.

* TOC
{:toc}

# **Cairo By Example (v0.10.0)**

## **Hello, World!**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
%builtins range_check

// Alphabet substituation cipher for each letter.
// a = 01, b = 02, etc.
const hello = 10000805121215;  // 08, 05, 12, 12, 15
const world = 10002315181204;  // 23, 15, 18, 12, 04.

// View decorator exposes the hello_world function for reading
// Returns two constants with the type felt
@view
func hello_world() -> (num_1: felt, num_2: felt) {
    return (hello, world);
}
```

## **Data Structures**

### **Felt Data Types**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
%builtins range_check
// A felt is an integer in the range of 0 <= x < P
// where P is a prime (i.e. P = 2^251 + 17 * 2^192 + 1).
// All computations are performed modulo P.
@view
func types(user_number: felt) -> (
    user_number_echoed: felt, string_literal: felt, mangled_string: felt, hello_string: felt
) {
    // In ASCII, a=0x6162 and b = 0x6163
    // 'ab' = 0x6162 = 24930
    let string_literal = 'ab';

    // 24930 + 1 = 24931 = 0x6163 = ab:61 c:63 = 'ac'
    let mangled_string = string_literal + 1;

    // h 68 e 65 l 6c l 6c o 6f
    // 0x68656c6c6f = 448378203247
    let hello_string = 'hello';

    return (user_number, string_literal, mangled_string, hello_string);
}

```
### **Variables**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// Constant variables defined here are available to all functions

// All persistent state appears in @storage_var
@storage_var
func persistent_state() -> (res: felt) {
}

@external
func use_variables{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    // Required for 'local' variables
    alloc_locals;

    // Revocable felt reference
    let my_ref = 50;
    // Redefine reference
    let my_ref = 51;

    // Revocable expression (temporary variable)
    tempvar my_temp = 2 * my_ref;
    // Redefine tempvar
    tempvar my_temp = 3 * my_ref;

    // Non-revocable felt (constant variable)
    const my_const = 60;
    // Cannot redefine const

    // Non-revocable expression (local). Requires alloc_locals
    local my_local = 70;
    // Cannot redefine local

    // Persistent @storage_var storage, without a variable
    persistent_state.write(80);
    // Redefine state
    persistent_state.write(81);

    // A variable can be assigned to a function output
    // let (my_var) = func()
    let (my_ref_2) = persistent_state.read();
    let (local my_local_2) = persistent_state.read();

    assert my_ref_2 = 81;
    assert my_local_2 = 81;
    return ();
}
```

### **Tuples**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet

from starkware.cairo.common.alloc import alloc
from starkware.cairo.common.registers import get_fp_and_pc

@view
func read_tuple() -> (value: felt) {
    alloc_locals;
    // Define a tuple
    local felt_tuple: (felt, felt, felt, felt) = (9, 8, 7, 18);

    // Access the tuple at the chosen index
    let val = felt_tuple[3];
    return (val,);
}

@external
func pass_tuple(index_1: felt, index_2: felt) -> (sum: felt) {
    // Tuple is passed by sending the address of the tuple
    alloc_locals;

    // Define a tuple
    local the_tuple: (felt, felt, felt, felt) = (4, 6, 8, 13);

    // Get the value of the fp register
    let (__fp__, _) = get_fp_and_pc();
    // The address of the tuple is sent using &tuple
    let (total) = get_sum(&the_tuple, index_1, index_2);
    return (total,);
}

func get_sum(tuple_ptr: felt*, idx_1: felt, idx_2: felt) -> (total: felt) {
    let sum = tuple_ptr[idx_1] + tuple_ptr[idx_2];
    return (sum,);
}
```

### **Arrays**

**DESCRIPTION**
- Arrays are defined using a pointer to the first element of the array.
- Their values are addressed by their location in memory relative to the pointer

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
%builtins range_check

// The alloc is used to return a newly allocated memory segment
from starkware.cairo.common.alloc import alloc

@view
func read_array{range_check_ptr}(index: felt) -> (value: felt) {
    // A pointer to the start of the array
    let (felt_array: felt*) = alloc();

    // [felt_array] is the value at the pointer
    // 'assert' sets the value at the index
    assert [felt_array] = 0;
    assert [felt_array + 1] = 1;
    assert [felt_array + 2] = 1;
    assert [felt_array + 2] = 2;
    assert [felt_array + 9] = 21;  // Sets index 9 to value 21

    // Access list at selected index
    let value = felt_array[index];
    return (value=value);
}
```

### **Structs**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

struct User {
    id_number: felt,
    is_admin: felt,
    vote_count: felt,
}

@storage_var
func admin_votes(user: felt) -> (count: felt) {
}

@view
func query_user{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(user_id: felt) -> (
    value: felt
) {
    let (votes) = admin_votes.read(user_id);
    return (votes,);
}

@external
func register_user{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    id: felt, admin: felt, votes: felt
) {
    alloc_locals;
    // Struct constructor is used to declare values
    local new_user: User = User(id_number=id, is_admin=admin, vote_count=votes);
    // Struct values can be changed with
    // 'assert new_user.is_admin = 0'
    if (new_user.is_admin == 1) {
        admin_votes.write(new_user.id_number, new_user.vote_count);
        return ();
    }
    return ();
}

```

### **Mappings**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// A single value
@storage_var
func store_a() -> (count: felt) {
}

// A single mapping of key -> value
@storage_var
func store_b(id_number: felt) -> (count: felt) {
}

// Nested mapping of key1 and key2 -> value
// Specific item size and item price combination -> count
@storage_var
func store_c(size: felt, price: felt) -> (count: felt) {
}

@external
func record_inventory{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    a_val: felt, b_id: felt, b_val: felt, c_size: felt, c_price: felt, c_val: felt
) {
    store_a.write(a_val);
    store_b.write(b_id, b_val);
    store_c.write(c_size, c_price, c_val);
    return ();
}

@view
func read_inventory{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    b_id: felt, c_size: felt, c_price: felt
) -> (storage_a: felt, storage_b: felt, storage_c: felt) {
    alloc_locals;
    // Read the requested parts of each inventory.
    // 'let (local xyz)'= func() is used for function return values

    let (local count_a) = store_a.read();
    let (local count_b) = store_b.read(b_id);
    let (local count_c) = store_c.read(c_size, c_price);

    return (count_a, count_b, count_c);
}

```

### **Dictionaries**

**DESCRIPTION**
- Get a pointer to a new dictionary with `default_dict_new()`
- Assert its integrity with `default_dict_finalize()`
- Assign a value to a key `dict_write()`
- Read the value of a key `dict_read()`

```rust
%lang starknet
%builtins range_check

from starkware.cairo.common.default_dict import default_dict_new, default_dict_finalize
from starkware.cairo.common.dict import dict_write, dict_read, dict_update

// Returns the value for the specified key in a dictionary
@view
func get_value_of_key{range_check_ptr}(key_1: felt, key_2: felt, key_3: felt) -> (
    val_1: felt, val_2: felt, val_3: felt
) {
    alloc_locals;
    // First create an empty dictionary and finalize it.
    // All keys will initially have value 25 {key: 25}
    let initial_value = 25;
    let (local dict) = default_dict_new(default_value=initial_value);
    // Finalize the dictionary. This ensures default value is correct.
    default_dict_finalize(
        dict_accesses_start=dict, dict_accesses_end=dict, default_value=initial_value
    );

    // Then add {key: val} pairs
    dict_write{dict_ptr=dict}(key=5, new_value=20);  // {5:20}
    dict_write{dict_ptr=dict}(key=10, new_value=10);  // {10: 10}

    // Check {key: value} pair is correct
    let (key_5_value) = dict_read{dict_ptr=dict}(key=5);

    // Update a key
    dict_update{dict_ptr=dict}(key=5, prev_value=20, new_value=5);  // {5: 20} -> {5: 5}

    // Check that an unused key returns the default value
    let (unused_key_999_val) = dict_read{dict_ptr=dict}(key=999);
    assert unused_key_999_val = 25;

    // Get value of the requested keys
    let (val_1) = dict_read{dict_ptr=dict}(key_1);
    let (val_2) = dict_read{dict_ptr=dict}(key_2);
    let (val_3) = dict_read{dict_ptr=dict}(key_3);

    return (val_1, val_2, val_3);
}
```
## **Returning Data Structures**

### **Array Returns**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func stored_number() -> (res: felt) {
}

// Function to get the stored value
@view
func get{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (stored: felt) {
    let (stored) = stored_number.read();
    return (stored,);
}

// Function to accept an array
@external
func save{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    input_array_len: felt, input_array: felt*
) {
    let first = input_array[0];
    let last = input_array[input_array_len - 1];
    let solution = first * 10 - last * 2;
    stored_number.write(solution);
    return ();
}
```

### **Struct Returns**

**DESCRIPTION**

#### **Struct Returns User Database**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

struct User {
    upvotes: felt,
    downvotes: felt,
    rank: felt,
}
@storage_var
func user_upvotes(user_id: felt) -> (count: felt) {
}

@storage_var
func user_downvotes(user_id) -> (count: felt) {
}

@storage_var
func user_rank(user_id) -> (count: felt) {
}

@external
func query_user{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(user_id: felt) -> (
    user_stats: User
) {
    let (up) = user_upvotes.read(user_id);
    let (down) = user_downvotes.read(user_id);
    let (rank) = user_rank.read(user_id);
    let user_stats = User(upvotes=up, downvotes=down, rank=rank);
    return (user_stats,);
}

@external
func register_user{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    user_id: felt, upvotes: felt, downvotes: felt, rank: felt
) {
    user_upvotes.write(user_id, upvotes);
    user_downvotes.write(user_id, downvotes);
    user_rank.write(user_id, rank);
    return ();
}

```

#### **Struct Returns User Analyst**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// The struct being received is also defined in the contract
struct User {
    upvotes: felt,
    downvotes: felt,
    rank: felt,
}

// The interface for the other function
@contract_interface
namespace IUserDatabase {
    func query_user(user_id: felt) -> (user_stats: User) {
    }
}

@external
func score_user{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    database_address: felt, user_id: felt
) -> (user_score: felt) {
    let (user: User) = IUserDatabase.query_user(database_address, user_id);
    let score = user.upvotes - user.downvotes;
    return (score,);
}
```

## **Setters and Getters**

### **Assert** 

**DESCRIPTION**

```rust
// An assert statement can be used for two purposes
// Checking the value of two variables are the same
// Setting the value of a variable that currently has no value

// Declare this file as a StarkNet contract.
%lang starknet

struct Registry {
    val_0: felt,
    val_1: felt,
}

@view
func asserter(test_0: felt, test_1: felt) -> (val_1: felt, val_2: felt) {
    alloc_locals;
    // Define a new instance of Registry struct
    local newRegistry: Registry;
    // Set value of a member
    assert newRegistry.val_0 = test_0;
    // Assert that the value is something else
    // Will fail unless 66 is chosen for test_0
    assert newRegistry.val_0 = 66;

    // Set the other value
    assert newRegistry.val_1 = test_1;

    // Assert that the two members do not have the same value
    assert newRegistry.val_0 = newRegistry.val_1;
    return (newRegistry.val_0, newRegistry.val_1);
}
```

### **Read and Write**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// Creating a variable called 'count' that stores a felt
@storage_var
func balance() -> (res: felt) {
}

// Function to retrieve current balance
@view
func get{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (value: felt) {
    let (value) = balance.read();
    return (value,);
}

// Function to update the stored number (felt)
@external
func save{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(input: felt) {
    balance.write(input);
    return ();
}
```

### **Read and Write Tuples**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// Creating a variable called 'count' that stores a felt
@storage_var
func stored_felt() -> (res: (felt, felt, felt)) {
}

// Function to retrieve stored tuple
@view
func get{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    res_1: felt, res_2: felt, res_3: felt
) {
    let (stored_tuple) = stored_felt.read();
    let res_1 = stored_tuple[0];
    let res_2 = stored_tuple[1];
    let res_3 = stored_tuple[2];

    return (res_1=res_1, res_2=res_2, res_3=res_3);
}

// Function to updaye the stored number (felt)
@external
func save{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    input_1: felt, input_2: felt, input_3: felt
) {
    stored_felt.write((input_1, input_2, input_3));
    return ();
}
```

### **Read and Write Arrays**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func stored_number() -> (res: felt) {
}

// Function to get the stored value
@view
func get{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (stored: felt) {
    let (stored) = stored_number.read();
    return (stored,);
}

// Function to accept an array
@external
func save{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    input_array_len: felt, input_array: felt*
) {
    let first = input_array[0];
    let last = input_array[input_array_len - 1];
    let solution = first * 10 - last * 2;
    stored_number.write(solution);
    return ();
}
```

## **Data Locations**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.alloc import alloc

// Memory
const a_contract_constant = 5;

// All persistent state appears inside @storage_var
@storage_var
func persistent_state() -> (res: felt) {
}

@external
func data_locations{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    alloc_locals;

    // Storage (@storage_var)
    persistent_state.write(15);

    // Memory
    let (an_array: felt*) = alloc();
    assert [an_array] = 30;
    assert [an_array + 1] = 60;
    local a_tuple: (felt, felt, felt) = (100, 200, 300);
    let a_reference = 500;
    tempvar a_temporary = 2 * a_reference;
    const a_function_constant = 1500;
    local a_local = 2000;
    let (a_state_reference) = persistent_state.read();
    let (local a_local_reference) = persistent_state.read();

    // The function could return any one of the above variables.
    return ();
}

@view
func read_values{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    val_1: felt, val_2: felt
) {
    // Only the Storage is available to someone calling this contract
    // Of all variables in this contract, only one is available
    let (val1) = persistent_state.read();
    let val2 = a_contract_constant;
    return (val1, val2);
}
```

### **Pointers**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet

from starkware.cairo.common.registers import get_fp_and_pc

// Pointers are used to communicate a data structure between functions.
// Rather than passing a tuple, a function can pass a pointer to the start of the tuple in memory.
@view
func use_pointer(number: felt) -> (res: felt) {
    // Variable 'tuple' is assigned to the pointer to the tuple, which is returned by the function.
    // This variable is of type felt*.
    let (tuple) = tuple_maker(number);
    // The variable 'val' is set to the value of the third element
    let val = tuple[2];
    return (val,);
}

// This function returns a pointer to a tuple
func tuple_maker(val: felt) -> (a_tuple: felt*) {
    alloc_locals;
    // Declare a tuple
    local tuple: (felt, felt, felt) = (5, 6, 2 * val);
    // Local variables are based on the frame pointer,
    // which can be accessed using the following library
    let (__fp__, _) = get_fp_and_pc();
    // & denotes the address, in this case, the address of the tuple
    return (&tuple,);
}

```
## **Function Decorators**

### **Constructors**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func stored_parameter() -> (res: felt) {
}

@storage_var
func important_contract_address() -> (res: felt) {
}

@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    special_number: felt, important_address: felt
) {
    stored_parameter.write(special_number);
    important_contract_address.write(important_address);
    return ();
}

// Func to get numbers stored at deployment
@view
func read_special_values{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    number: felt, address: felt
) {
    let number = stored_parameter.read();
    let address = important_contract_address.read();
    return (number, address);
}

```

### **Function Visibility**

**DESCRIPTION**
- Functions with a decorator (@view, @external @storage) only handles felt type arguments
- Generic helper functions can be used to handle arguments other than felt

- Contracts have 2 entry points, where generic functions or storage may be accessed
    - @external for writing -> @storage to write state, or use a generic helper function
    - @view for reading -> @storage to read state, or use a generic helper function

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.alloc import alloc

struct dataStruct {
    a: felt,
    b: felt,
}

// Only felt args
@storage_var
func storage() -> (res: felt) {
}

// Only felt args
@external
func write{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(val: felt) {
    helper_1(val);
    // Write functions are a transaction and should not return values
    return ();
}

// Only felt args
@view
func read{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    val_1: felt, val_2: felt, val_3: felt
) {
    // Brackets around val_2 to recieve the value from helper_2()
    let (val_2) = helper_2();
    let (stored_val) = storage.read();
    // All fields in a tuple must have a name
    return (val_1=3, val_2=val_2, val_3=stored_val);
}

// Other args, including felt allowed
func helper_1{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(val: felt) {
    // Implicit args in curly bracketsa re required, such as for storage related pointers
    storage.write(val);
    // Function does not return a value;
    return ();
}

// Return values are designated by an arrow ->
func helper_2() -> (val: felt) {
    // Function accepts 0 args, and returns 1. No implicit args requires.
    // data variable is assigned to a struct
    let data = dataStruct(a=4, b=5);
    // Struct is passed as argument
    let (processed_data) = helper_3(data);
    // Use of the name of a return var is optional
    return (processed_data,);
}

func helper_3(a_b_data: dataStruct) -> (processed_data: felt) {
    // Function accepts a pointer (to dataStruct instance) as seen by the '*' astrix
    // No implicit args required. A 'tempvar' variable is needed for the compound expression
    tempvar a_b_product = a_b_data.a * a_b_data.b;
    return (processed_data=a_b_product,);
}

```

## **Control Structures**

### **If Else**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// First Criteria
@storage_var
func criterion() -> (val: felt) {
}

// Second Criteria
@storage_var
func criterion2() -> (val: felt) {
}

// Total sum of values that met criteria
@storage_var
func met_criteria_total() -> (val: felt) {
}

// Fetch total sum value
@view
func match_count{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    count: felt
) {
    let (num) = met_criteria_total.read();
    return (num,);
}

@external
func is_match{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(number: felt) -> (
    result: felt
) {
    let (condition) = criterion.read();
    if (number == condition) {
        let (total) = perform_function_single();
        return (total,);
    } else {
        return (0,);
    }
}

// If statements can optionally be followed by an else
// Both if and else must contain a return statement
// Else if is not supported
@external
func is_match_two_conditions{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    number1: felt, number2: felt
) -> (result: felt) {
    alloc_locals;
    let (local condition1) = criterion.read();
    let (local condition2) = criterion2.read();

    // Boolean AND condition to ensure number1 is not equal to condition2
    // and that number2 is not equal to condition1
    if (number1 == condition2 and number2 == condition1) {
        return (0,);
    }

    // Else blocks are not supported with the AND boolean logic expressions yet.
    // if (number1 == condition1 and number2 == condition2) {
    //    let (total) = perform_function();
    //    return (total,);
    // } else {
    //    return (0,);
    // }

    // To combat this, nested loops can be used to achieve the same effect
    if (number1 == condition1) {
        if (number2 == condition2) {
            let (total) = perform_function_total();
            return (total,);
        } else {
            return (0,);
        }
    }
    return (0,);
}

// Write value to storage variable
@external
func store_criterion{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    number1: felt
) {
    criterion.write(number1);
    return ();
}

// Write values to storage variables
@external
func store_two_criterion{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    number1: felt, number2: felt
) {
    criterion.write(number1);
    criterion2.write(number2);
    return ();
}

// Function has no decorator - No direct user interaction
// Perform a summation of previous values and single criteria
func perform_function_single{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    total: felt
) {
    let (mem) = met_criteria_total.read();
    let (crit) = criterion.read();
    met_criteria_total.write(mem + crit);
    return (mem + crit,);
}

// Function has no decorator - No direct user interaction
// Perform a summation of previous values and both the criteria
func perform_function_total{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    total: felt
) {
    let (mem) = met_criteria_total.read();
    let (crit) = criterion.read();
    let (crit2) = criterion2.read();

    let total = crit + crit2;
    met_criteria_total.write(mem + total);
    return (mem + total,);
}

```

### **Recursive Loops**

**DESCRIPTION**
- Steps to Recursive Loops
    - Specify loop length
    - A looping function is called with a list of elements
    - It checks if the element if the final one, if not it increments and calls itself.
    - The previous step is repeated until the final element is reached
    - The function continues to execute the desired operation
    - The second last element is reached, then the third last, right until the first element
    - The end of the function is reaches
    - The result is returned to the calling function

```rust
// Declare this file as a StarkNet contract.
%lang starknet

from starkware.cairo.common.alloc import alloc

@view
func read_sum() -> (sum: felt) {
    let (array: felt*) = alloc();
    assert [array] = 2;
    assert [array + 1] = 24;
    assert [array + 2] = 4;
    let (sum) = get_sum(array, 3);
    return (sum,);
}

// Function has no decorator - No direct user interaction
func get_sum(array: felt*, length: felt) -> (sum: felt) {
    if (length == 0) {
        // Start with sum=0
        return (sum=0,);
    }

    let (current_sum) = get_sum(array=array + 1, length=length - 1);
    // This part of the function is first reached when length = 0
    // The sum begins. This is the sequence: 2, 24 + 2 then 4 + 26
    let sum = [array] + current_sum;
    // The return function targets the body of this function
    // 3 times before returning to the body of read_sum()
    return (sum,);
}

```

## **Imports**

**DESCRIPTION**

### **Custom Imports**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin
// Import a function from a custom local file
from utils.math_custom_import import add_two, get_modulo

// This is the main contract file that will be deployed.
@external
func get_calculations{range_check_ptr}(first: felt, second: felt) -> (sum: felt, modulo: felt) {
    // Two custom operations
    let (sum) = add_two(first, second);
    let (modulo) = get_modulo(first, second);
    return (sum, modulo);
}
```

### **Custom Math Library**

```rust
%lang starknet
// Function is from common library
from starkware.cairo.common.math import unsigned_div_rem

// This function is imported by 'custom_import.cairo'
// Equivalent to plaxing this function in the file
func add_two(a: felt, b: felt) -> (sum: felt) {
    let sum = a + b;
    return (sum,);
}

// This function performs modulo
func get_modulo{range_check_ptr}(a: felt, b: felt) -> (result: felt) {
    let (dividend, remainder) = unsigned_div_rem(a, b);
    // The divident is not used, and the following is equivalent:
    // let (_, remainder) = unsigned_div_rem(a, b)
    return (remainder,);
}

```

## **Interfaces**

**DESCRIPTION**

### **Contract Calls A**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// A stores a number
@storage_var
func number_in_A() -> (res: felt) {
}

// Address of contract B so that A can call B
@storage_var
func contract_B_address() -> (res: felt) {
}

// This makes this contract (A) aware of B
// Basically, copy the functions needed and strip out implicit arguments
@contract_interface
namespace contract_B {
    func increment(number: felt) {
    }

    func read_number() -> (number: felt) {
    }
}

// Function to get the stored number
@view
func get_AB_system_status{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    system_sum: felt
) {
    // Sums A and B stored numbers
    let (a_num) = number_in_A.read();
    // Fetch address of B from storage
    let (b_addr) = contract_B_address.read();
    let (b_num) = contract_B.read_number(b_addr);
    let res = a_num + b_num;
    return (res,);
}

// Function to update
@external
func update_AB_system{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    increment_for_A: felt, increment_for_B: felt
) {
    // Read A
    let (a_current) = number_in_A.read();
    // Add and save
    number_in_A.write(a_current + increment_for_A);
    // Read B by calling via interface
    let (b_addr) = contract_B_address.read();
    // Format for using interfaces:
    // contract_name.function(address, arg_1, arg_2, ...)
    contract_B.increment(b_addr, increment_for_B);
    return ();
}

// Save the address of contract B
@external
func set_B_address{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(address: felt) {
    // Save to store in A (this contract)
    // Now A knows where to find B
    // It is already aware of the nature if B,
    // due to the interface function
    contract_B_address.write(address);
    return ();
}

```

### **Contract Calls B**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin
// Function to retrieve caller_address
from starkware.starknet.common.syscalls import get_caller_address

// B stores a number
@storage_var
func number_in_B() -> (res: felt) {
}

// Address of contract A so that B can call A
@storage_var
func contract_A_address() -> (res: felt) {
}

// Run on deployment only. Must contain constructor in name and decorator
@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    address_of_contract_A: felt
) {
    // When this contract is deployed, passin it the known address of A
    // allows B to restrict write access
    contract_A_address.write(address_of_contract_A);
    // No way to modify this after deployment
    return ();
}

// Function to retrieve the stored number
@view
func read_number{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (res: felt) {
    // Anyone can call this view only function
    let (res) = number_in_B.read();
    return (res,);
}

@external
func increment{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(input: felt) {
    // When using locals, 'alloc_locals' is essential
    alloc_locals;
    // See who is attempting to modify B (this contract)
    // Locals cannot be updated, only redefined.
    let (local address) = get_caller_address();
    // Lookup address of A stored in the constructor
    let (known_address) = contract_A_address.read();
    // Make sure A is calling
    // assert x = y only works if x is local.
    // Otherwise, x is remapped to equal y.
    assert address = known_address;
    // Get the current stored number
    let (current) = number_in_B.read();
    // Add and save
    number_in_B.write(current + input);
    return ();
}

```

## **Math**

### **Math Library**

**DESCRIPTION**

- Not zero - `assert_not_zero(val)` : Asserts val is not zero.
- Not equal - `assert_not_equal(a, b)` : Asserts that a is not equal to b.
- Not negative - `assert_nn(val)` : Asserts that val is not negative.
- Less than or equal to - `assert_le(a, b)` : Asserts that a is less than or equal to b.
- Less than - `assert_lt(a, b)` : Asserts that a is less than b.
- Not negative and less than or equal to - `assert_nn_le()` : Asserts a is not negative and less than or equal to b
- In range - `assert_in_range(val, a, b)` : Asserts val is both greater than or equal to a, and less than b.
- 250-bit - `assert_250_bit(val)` : Asserts that val is smaller than the maxiumum value in 250-bit space and non negative
- Split felt - `split_felt(val)` : Returns high and low of parts of val.
- Less than or equal to with split felt - `assert_le_felt(a, b)` : Asserts a is less than or equal to b using split felt method
- Less than with split felt - `assert_lt_felt(a, b)` : Asserts a is less than b using split felt method
- Absolute value - `abs_value(val)` : Returns val as positive value
- Sign - `sign(val)` : Returns one of -1, 0 or 1 for a val that is negative, zero or positive respectively.
- Unsigned division remainder : `unsigned_div_rem(value, div)` : Returns the quotient q and remainder r from the integer division of value/div as positive values
- Signed division remainder : `signed_div_rem(value, div)` : Returns the quotient q and remainder r from the integer division of value/div, with quotient sign either positive or negative
    - Handles integer and modulo operations with negative numbers in the same way python does, where -100 // 3 = -43 & -100 % 3 = 2

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

from starkware.cairo.common.math import (
    assert_not_zero,
    assert_not_equal,
    assert_nn,
    assert_le,
    assert_lt,
    assert_nn_le,
    assert_in_range,
    assert_250_bit,
    split_felt,
    assert_le_felt,
    assert_lt_felt,
    abs_value,
    sign,
    unsigned_div_rem,
    signed_div_rem,
)

// Accepts two numbers for integer division (signed, unsigned)
// Demonstrates successful cases of math operations
@view
func check_values{range_check_ptr}(num_1: felt, num_2: felt) -> (
    u_quot: felt, u_rem: felt, s_quot: felt, s_rem: felt
) {
    alloc_locals;

    assert_not_zero(100);
    assert_not_zero(-100);
    // assert_not_zero(0); // Fails

    assert_not_equal(100, 150);
    // assert_not_equal(100, 100); // Fails

    assert_nn(100);
    assert_nn(0);
    // assert_nn(-1); //Fails

    assert_le(100, 150);
    assert_le(100, 100);
    // assert_le(100, 50); // Fails

    assert_lt(100, 150);
    // assert_lt(100, 100); // Fails
    // assert_lt(150, 100); // Fails

    assert_nn_le(100, 150);
    // assert_nn_le(-100, 150); // Fails
    // assert_nn_le(100, 50); // Fails

    assert_in_range(150, 100, 200);
    assert_in_range(150, -100, 200);
    // assert_in_range(50, 100, 200); // Fails

    assert_250_bit(9234);
    assert_250_bit(2 ** 250 - 1);
    // assert_250_bit(2**250); // Fails
    // assert_250_bit(-100); // Fails

    let (high_1, low_1) = split_felt(100 * 2 ** 128 + 150);
    assert high_1 = 100;  // Just crosses mid point (128 bits)
    assert low_1 = 150;

    let (high_2, low_2) = split_felt(100 * 2 ** 130 + 150);
    assert high_2 = 100 * 2 ** 2;  // Slightly more than high_1
    assert low_2 = 150;

    assert_le_felt(150, 200);
    assert_le_felt(150, 150);
    // assert_le_felt(150, 50); // Fails

    assert_lt_felt(150, 200);
    // assert_lt_felt(150, 150); // Fails
    // assert_lt_felt(150, 50); // Fails

    local a = abs_value(-150);
    assert a = 150;

    local b = sign(0);
    assert b = 0;

    local c = sign(-50);
    assert c = -1;

    let (local d, e) = unsigned_div_rem(100, 3);
    assert d = 33;
    assert e = 1;

    let (local u_quot, u_rem) = unsigned_div_rem(num_1, num_2);

    // This check must be less than 2 ** 64
    let RANGE_CHECK_BOUND = 2 ** 20;

    let (local f, g) = signed_div_rem(-100, 3, RANGE_CHECK_BOUND);
    // To have these asserts pass, the -1 and +1 have been added to the expected values
    assert f = -34;
    assert g = 3;

    let (local s_quot, s_rem) = signed_div_rem(-num_1, num_2, RANGE_CHECK_BOUND);

    return (u_quot, u_rem, s_quot, s_rem);
}

```

### **Math Comparators**

**DESCRIPTION**

- Not Zero - `is_not_zero(val)` : Checks if val is not zero
- Not Negative - `is_nn(val)` : Checks if val is not negative
- Not Negative and less than or equal to - `is_nn_le(val)` : Checks if val is not negative and is less than or equal to a
- Less than or equal to - `is_le(val, a)` : Checks if val less than or equal to a
- In range - `is_in_range(val, a, b)` - Checks if val larger than or equal to a and smaller than or equal to b
- Less than or equal to for felts - `is_le_felt(a, b)` : Checks if a_high is less than b_high, obtained using split_felt(val)

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

from starkware.cairo.common.math_cmp import (
    is_not_zero,
    is_nn,
    is_le,
    is_nn_le,
    is_in_range,
    is_le_felt,
)

@view
func check_values{range_check_ptr}(number: felt) -> (
    a: felt, b: felt, c: felt, d: felt, e: felt, f: felt
) {
    alloc_locals;

    // 1 if number is not zero
    local a = is_not_zero(number);

    // 1 if number - 100 not negative
    // (1 if number greater than or equal to 100)
    local b = is_nn(number - 100);

    // 1 if number less than or equal to 100
    local c = is_le(number, 100);

    // 1 if number is not negative and is less than or equal to 100
    local d = is_nn_le(number, 100);

    // 1 if number is greater than or equal to 100 and less than 250
    local e = is_in_range(number, 100, 250);

    // 1 if number is less than equal to 100, using integer lift to first compare the HIGHS, and if needed
    // then check the LOWS. For exa,ple:
    // number = num_HIGH * 2 ** 128 + num_LOW
    // another_number = num_2_HIGH * 2 ** 128 + num_2_LOW
    let another_number = 100;
    local f = is_le_felt(number, another_number);

    return (a, b, c, d, e, f);
}
```

### **Counter**

**DESCRIPTION**

- The three implicit arguments are required for storage operations.
- When returning function call values to a reference, ensure the reference is inside brackets.

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// Creating a variable called 'count' that stores a felt
@storage_var
func balance() -> (res: felt) {
}

// Function to retrieve current balance
@view
func get{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (value: felt) {
    let (value) = balance.read();
    return (value,);
}

// Function to increment balance by 1
@external
func increment{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    let (res) = balance.read();
    balance.write(res + 1);
    return ();
}

// Function to decrement balance by 1
@external
func decrement{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    let (res) = balance.read();
    balance.write(res - 1);
    return ();
}
```

### **Currency**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// Creating a variable called stores user and balance to a felt
@storage_var
func wallet_balance(user: felt) -> (res: felt) {
}

@external
func register_currency{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    user: felt, register_amount: felt
) {
    alloc_locals;
    let (local balance) = wallet_balance.read(user);
    wallet_balance.write(user, balance + register_amount);
    return ();
}

@external
func move_currency{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    from_user: felt, to_user: felt, move_amount: felt
) {
    alloc_locals;
    let (local sender_balance) = wallet_balance.read(from_user);
    let (local receiver_balance) = wallet_balance.read(to_user);
    wallet_balance.write(to_user, receiver_balance + move_amount);
    wallet_balance.write(from_user, sender_balance - move_amount);
    return ();
}

@view
func check_wallet{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(user: felt) -> (
    balance: felt
) {
    alloc_locals;
    let (local balance) = wallet_balance.read(user);
    return (balance,);
}

```
## Bitwise

### **Bitwise Operations**

**DESCRIPTION**
- `bitwise_and(x, y)` - the result of bitwise AND operation on x and y
- `bitwise_xor(x, y)` - the result of bitwise XOR operation on x and y
- `bitwise_or(x, y)` - the result of bitwise OR operation on x and y

```rust
%builtins bitwise
from starkware.cairo.common.cairo_builtins import BitwiseBuiltin
from starkware.cairo.common.bitwise import bitwise_operations, bitwise_and, bitwise_xor, bitwise_or

func main{bitwise_ptr: BitwiseBuiltin*}() -> () {
    // Define two binary numbers, a and b, using powers of 2 representation

    // Binary: a = 1100
    let a = 1 * 2 ** 3 + 1 * 2 ** 2 + 0 * 2 ** 1 + 0 * 2 ** 0;
    assert a = 1 * 8 + 1 * 4 + 0 * 2 + 0 * 1;
    assert a = 12;  // Decimal representation

    // Binary: b = 1010
    let b = 1 * 2 ** 3 + 0 * 2 ** 2 + 1 * 2 ** 1 + 0 * 2 ** 0;
    assert b = 1 * 8 + 0 * 4 + 1 * 2 + 0 * 1;
    assert b = 10;  // Decimal representation

    // 1100 AND 1010 = 1000
    let (a_and_b) = bitwise_and(a, b);
    assert a_and_b = 1 * 2 ** 3 + 0 * 2 ** 2 + 0 * 2 ** 1 + 0 * 2 ** 0;

    // 1100 XOR 1010 = 0110
    let (a_xor_b) = bitwise_xor(a, b);
    assert a_xor_b = 0 * 2 ** 3 + 1 * 2 ** 2 + 1 * 2 ** 1 + 0 * 2 ** 0;

    // 1100 XOR 1010 = 1110
    let (a_or_b) = bitwise_or(a, b);
    assert a_or_b = 1 * 2 ** 3 + 1 * 2 ** 2 + 1 * 2 ** 1 + 0 * 2 ** 0;

    // User defined values x and y. Returns all three operations
    let (_and, _xor, _or) = bitwise_operations(a, b);

    // Check that they match the result above
    assert _and = a_and_b;
    assert _xor = a_xor_b;
    assert _or = a_or_b;

    // Bitwise operations must be less than 251-bit
    // let (c) = bitwise_or(2**251, 2**3); // Fails
    let (c) = bitwise_or(2 ** 250, 2 ** 3);
    assert c = 2 ** 250 + 2 ** 3;

    return ();
}

```

### **Bitwise Starknet**

**DESCRIPTION**
- `bitwise_and(x, y)` - the result of bitwise AND operation on x and y
- `bitwise_xor(x, y)` - the result of bitwise XOR operation on x and y
- `bitwise_or(x, y)` - the result of bitwise OR operation on x and y

```rust
%lang starknet
%builtins bitwise

from starkware.cairo.common.cairo_builtins import BitwiseBuiltin
from starkware.cairo.common.bitwise import bitwise_operations, bitwise_and, bitwise_xor, bitwise_or

@view
func check_bitwise{bitwise_ptr: BitwiseBuiltin*}() -> () {
    // Define two binary numbers, a and b, using powers of 2 representation

    // Binary: a = 1100
    let a = 1 * 2 ** 3 + 1 * 2 ** 2 + 0 * 2 ** 1 + 0 * 2 ** 0;
    assert a = 1 * 8 + 1 * 4 + 0 * 2 + 0 * 1;
    assert a = 12;  // Decimal representation

    // Binary: b = 1010
    let b = 1 * 2 ** 3 + 0 * 2 ** 2 + 1 * 2 ** 1 + 0 * 2 ** 0;
    assert b = 1 * 8 + 0 * 4 + 1 * 2 + 0 * 1;
    assert b = 10;  // Decimal representation

    // 1100 AND 1010 = 1000
    let (a_and_b) = bitwise_and(a, b);
    assert a_and_b = 1 * 2 ** 3 + 0 * 2 ** 2 + 0 * 2 ** 1 + 0 * 2 ** 0;

    // 1100 XOR 1010 = 0110
    let (a_xor_b) = bitwise_xor(a, b);
    assert a_xor_b = 0 * 2 ** 3 + 1 * 2 ** 2 + 1 * 2 ** 1 + 0 * 2 ** 0;

    // 1100 XOR 1010 = 1110
    let (a_or_b) = bitwise_or(a, b);
    assert a_or_b = 1 * 2 ** 3 + 1 * 2 ** 2 + 1 * 2 ** 1 + 0 * 2 ** 0;

    // User defined values x and y. Returns all three operations
    let (_and, _xor, _or) = bitwise_operations(a, b);

    // Check that they match the result above
    assert _and = a_and_b;
    assert _xor = a_xor_b;
    assert _or = a_or_b;

    // Bitwise operations must be less than 251-bit
    // let (c) = bitwise_or(2**251, 2**3); // Fails
    let (c) = bitwise_or(2 ** 250, 2 ** 3);
    assert c = 2 ** 250 + 2 ** 3;

    return ();
}
```

## **Messages**

### **Generate Message**

**DESCRIPTION**
- Starknet contract can message L1 using the send_message_to_L1() function containing the arguments `to_address`, `payload_size` and `payload`.

- This can be recieved by calling the L1 Starkent contract function `consumeMessageFromL2()` from the addressed specified in the `to_address` above containing the arguments `from_address` (L2 address) and `payload`.

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins range_check

from starkware.cairo.common.alloc import alloc
from starkware.starknet.common.messages import send_message_to_l1

// Generates a message that an L1 contract can use
@external
func generate{syscall_ptr: felt*, range_check_ptr}() {
    // Messages require the syscall_ptr

    // A message is sent as a pointer to a list
    let (message: felt*) = alloc();
    assert message[0] = 444;
    assert message[1] = 333;

    // Send the message with the required argumentds
    send_message_to_l1(to_address=0x123123, payload_size=2, payload=message);
    return ();
}

```

### **Send Message to L1**

**DESCRIPTION**

StarkNet (SN) contract can specify a message for an L1 Ethereum (ETH) contract to recieve.
Three steps to this - generate, verify and digest
STEPS:
- 1: Custom contract on SN : Generates message -> application specific contract containing `send_message_to_l1()`
- 2: SN contract on ETH : Verifies message validity -> StarkNet.sol contains function `consumeMessageFromL2()` -> computes the hash that ties the messages to both L1 and L2 contracts -> checks hash is stored as a fact, verified by STARK validity proof
- 3: Custom contract on ETH : Digests message -> L1L2Example.sol contains `withdraw()` function. -> this calls `StarkNet.sol` and verifies that the payload is valid from the specified L2 address.

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.alloc import alloc
from starkware.starknet.common.messages import send_message_to_l1

// Demo contract 'L1L2Contract.sol' Starkware deployed to Ropsten
const L1_CONTRACT_ADDRESS = (0xce08635cc6477f3634551db7613cc4f36b4e49dc);

@storage_var
func stored_felt() -> (res: felt) {
}

// Generate a message than an L1 contract can use
@external
func increase_L1_balance{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    // Messages use the syscall_ptr

    // Create a dynamically sized array for the message
    let (message: felt*) = alloc();

    // The L1 contract expects the message to have 3 elements
    assert message[0] = 0;  // 0 is for withdrawl L2 -> L1
    assert message[1] = 12345678;  // User
    assert message[2] = 3;  // Amount to increase L1 balance by

    // Send the message
    send_message_to_l1(to_address=L1_CONTRACT_ADDRESS, payload_size=3, payload=message);
    return ();
}

```

### **Recieve Message**

**DESCRIPTION**
- StarkNet contract can receive an L1 message using the @l1_handler
- The recieving function is 'actioned' by the Starknet sequencer and then recieves the arguments `from_address` (l1 contract address thag sent the message) and message elements as type felt.

- The message originates on L1 with a call to Starknet contract function sendMessageToL2() with the arguments `to_address` (L2 address), `selector` and `payload`.

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin

// Stores the sum
@storage_var
func message_sum() -> (res: felt) {
}

// Recieve an L1 message. Sequencer actions this function
@l1_handler
func recieve{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    from_address: felt, message_index_0: felt, message_index_1: felt
) {
    // Add the two parts of the message together and save
    tempvar sum = message_index_0 + message_index_1;
    message_sum.write(sum);
    return ();
}
```

## **Cryptography**

### **Pedersen Hash**

**DESCRIPTION**

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
// Pedersen will allow us to use the Pedersen hash function native to many operations
%builtins pedersen range_check

// The HashBuiltin type is required when passing a pedersen_ptr as an implicit argument
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.hash import hash2

@view
func get_hash{pedersen_ptr: HashBuiltin*}(x, y) -> (hash: felt, hash_with_zero: felt) {
    // Pedersen hash of x and y
    // The hash function used will be the pointer that is passed
    let (hash) = hash2{hash_ptr=pedersen_ptr}(x, y);

    // Hash of x and 0. Equivalent of the hash of x alone
    let (hash_with_zero) = hash2{hash_ptr=pedersen_ptr}(x, 0);

    return (hash, hash_with_zero);
}
```

### **Verify ECDSA**

**DESCRIPTION**
- Cairo has a bultin to perform ECDSA signature verification.
- STEPS:
    - 1: Create a message to sign
    - 2: hash message using pedersen function 
    - 3: obtain private key 
    - 4: sign message hash using priv key 
    - 5: record sig_r & sig_s

- A Python script can be used to generate private key but in this case, we are using the hardcoded values.

- `message_hash` ("13579") : 2255487090060981340412778270523682108366421523848318719210969003889439916982
- `public_key` : 336651705928807190204460653884405974785205047669862626875647782176669707088
- `signature_r` : 1893532933103991730127061797833157421284043895315515223059211080812570729772
- `signature_s` : 2767847065606260480373088228849935790434610797527549053315487023265903912514

```rust
// Declare this file as a StarkNet contract.
%lang starknet
// ecdsa is a builtin that tracks the signature in the cairo trace.
%builtins ecdsa

from starkware.cairo.common.signature import verify_ecdsa_signature
// SignatureBuiltin is a struct used to represend the signature
from starkware.cairo.common.cairo_builtins import SignatureBuiltin

@view
func check_signature{ecdsa_ptr: SignatureBuiltin*}(message_hash, public_key, sig_r, sig_s) -> () {
    // If the signature is incorrect, this will fail
    verify_ecdsa_signature(message_hash, public_key, sig_r, sig_s);
    return ();
}
```

# **Cairo Security By Example (v0.10.0)**

## **Missing Input Validation**

**DESCRIPTION**
- In StarkNet, it is important to ensure that any user input from external functions are validated correctly.
- This includes checking that all types, which includes felt values and uint256 values, are within the allowed range.
- Values should be checked to ensure that they are within an acceptable boundary to ensure the protocol wont behave unexpectedly.
- Although there are definitely cases where using zero or negative values is appropriate, developer discretion should be used.

```rust
@external
func insecure_token_initializer{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    _admin: felt,
    _market: felt,
    _underlying_token: felt,
    _token_name: felt,
    _token_symbol: felt,
    _token_decimals: felt,
) {
    admin.write(_admin);
    market.write(_market);
    underlying.write(_underlying_token);
    token_name.write(_token_name);
    token_symbol.write(_token_symbol);
    token_decimals.write(_token_decimals);

    return ();
}

@external
func more_secure_token_initializer{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    _admin: felt,
    _market: felt,
    _underlying_token: felt,
    _token_name: felt,
    _token_symbol: felt,
    _token_decimals: felt,
) {
    const TOKEN_DECIMALS = 18;

    with_attr error_message("Token: zero address for admin") {
        assert_not_zero(_admin);
    }
    admin.write(_admin);

    with_attr error_message("Token: zero address for market") {
        assert_not_zero(_market);
    }
    market.write(_market);

    with_attr error_message("Token: zero address for underlying") {
        assert_not_zero(_underlying_token);
    }
    underlying.write(_underlying_token);

    with_attr error_message("Token: zero value for token name") {
        assert_not_zero(_token_name);
    }
    token_name.write(_token_name);

    with_attr error_message("Token: zero value for token symbol") {
        assert_not_zero(_token_symbol);
    }
    token_symbol.write(_token_symbol);

    with_attr error_message("Token: zero value for token decimals") {
        assert_not_zero(_token_decimals);
    }
    with_attr error_message("Token: token decimals out of range") {
        assert_le_felt(_token_decimals, TOKEN_DECIMALS);
    }
    token_decimals.write(_token_decimals);

    return ();
}
```

**FIX**
- Ensure that all values are checked to be within expected ranges and not negative or zero unless required.

## **Account Abstraction Missing Zero Address Checks**

**DESCRIPTION**
- In StarkNet, the account abstraction model differs from the Solidity account model.
- There are no EOA accounts in StarkNet, which means that all addresses are contract addresses.
- Generally, when implementing a StarkNet account, users will deploy a contract that authenticates the user and makes calls on the users behalf.
- Although calls are expected to be performed through a contract account, it is still possible to directly send transactions to contracts directly.
- From the perspective of the contract, the syscall get_caller_address will always return zero.
- This could result in broken access control checks or even sending tokens to a zero address, accidentally burning them from the supply.

```rust
@external
func insecure_claim_tokens{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    sender_address: felt
) {
    let (user) = get_caller_address();
    // No validation on caller
    let (user_current_balance) = user_balances.read(sender_address);
    // Writes the updated balance to the address zero.
    // Someone may now directly call the contract and withdraw those funds.
    user_balances.write(user, user_current_balance + 200);

    return ();
}

@external
func more_secure_claim_tokens{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    sender_address: felt
) {
    let (user) = get_caller_address();
    assert_not_zero(user);

    let (user_current_balance) = user_balances.read(sender_address);
    user_balances.write(user, user_current_balance + 200);

    return ();
}
```

**FIX**
- Add zero address checks, which will prevent users from interacting directly with the contract.


## **Lack of Address Sanity Checks**

**DESCRIPTION**
- When inputting Ethereum addresses into Starknet for L1-L2 communication, you must validate the Ethereum address entered into Starknet DApp.
- Ensure the Ethereum address is bound checked, as if a user enters an invalid Ethereum address it could cause unexpected behaviour to occur when communicating with Ethereum layer.
- StarkNet addresses are of type felt, which is between the range of 0 < x < P (2^251 + 17 * 2^192 + 1), whereas L1 Ethereum addresses are of type uint160, which is less than the felt size.

```rust
@external
func insecure_set_l1_bridge{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    l1_bridge_address: felt
) {
    l1_bridge.write(val=l1_bridge_address);
    return ();
}

const ETH_ADDRESS_BOUND = 2 ** 160;

@external
func more_secure_set_l1_bridge{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    l1_bridge_address: felt
) {
    // Check l1 bridge address is valid
    with_attr error_message("Invalid L1 Ethereum address") {
        assert_lt_felt(l1_bridge_address, ETH_ADDRESS_BOUND);
        assert_not_zero(l1_bridge_address);
    }
    l1_bridge.write(val=l1_bridge_address);
    return ();
}
```

**FIX**
- When using the 160-bit Ethereum address in StarkNet, ensure that the L1 address is in the range 0 < X < 2**160

## **Interacting with Arbitrary Tokens**

**DESCRIPTION**
- When interacting with arbitrary tokens, it is essential to ensure that each transfer is validated with balance checks.
- This is because some tokens might not implement the same logic on transfer.
- It is recommended to check the balance of the token before, and after the transfer to ensure that the amount transferred matches.

```rust
@external
func insecure_pay_someone{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    amount: Uint256, reciever: felt, token: felt
) {
    let (sender) = get_caller_address();
    IERC20.transferFrom(contract_address=token, sender=sender, recipient=reciever, amount=amount);
    return ();
}

@external
func more_secure_pay_someone{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    amount: Uint256, reciever: felt, token: felt
) {
    let (sender) = get_caller_address();
    let (balance_before) = IERC20.balanceOf(contract_address=token, account=reciever);
    IERC20.transferFrom(contract_address=token, sender=sender, recipient=reciever, amount=amount);
    let (balance_after) = IERC20.balanceOf(contract_address=token, account=reciever);
    let (calculated_balance: Uint256) = SafeUint256.add(balance_before, amount);
    let (is_equal) = uint256_eq(balance_after, calculated_balance);
    with_attr error_message("Incorrect balance") {
        assert is_equal = 1;
    }
    return ();
}
```

**FIX**
- Calculate the expected balance increase and then compare whether the updated balance after the transfer matches what is expected.

## **Uint256 Checks**

**DESCRIPTION**
- Uint256 values are made up from two felt, which should contain 128-bits each.
- This means that it is possible to control both the lower and upper portions of a Uint256.
- This enables malicious actors to manipulate the contract logic for their own gain.
- Code should always verify that any Uint256 arguments are valid Uint256, using the uint256_check,

```rust
@external
func insecure_send_message{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    message_data: Uint256
) {
    let (contract_address) = l1_contract.read();
    let (message_payload: felt*) = alloc();
    // No checks performed on the size of messa
    assert message_payload[0] = message_data.low;
    assert message_payload[1] = message_data.high;

    send_message_to_l1(to_address=contract_address, payload_size=2, payload=message_payload);
    return ();
}

@external
func more_secure_send_message{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    message_data: Uint256
) {
    let (contract_address) = l1_contract.read();
    let (message_payload: felt*) = alloc();

    with_attr error_message("Invalid Uint256") {
        uint256_check(message_data);
    }

    assert message_payload[0] = message_data.low;
    assert message_payload[1] = message_data.high;

    send_message_to_l1(to_address=contract_address, payload_size=2, payload=message_payload);
    return ();
}
```

**FIX**
- When using a Uint256, use the uint256_check from the Starknet library Uint256 to ensure it is a valid Uint256 value.

## **Incorrect Felt Comparison**

**DESCRIPTION**
- Cairo has two different methods to check whether a value is less than or equal to the operator; 'assert_le' and 'assert_nn_le'. 'assert_le' verifies that a <= b regardless of the size of a. 'assert_nn_le' verifies that  0 <= a <= b.
- This means 'assert_nn_le' will assert that a is non-negative as well (i.e. not greater than the RANGE_CHECK_BOUND of 2^128)

```rust
@storage_var
func max_supply() -> (res: felt) {
}

@external
func bad_comparison{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    let (value: felt) = ERC20.total_supply();
    // Check may allow for negative supply balance
    assert_le{range_check_ptr=range_check_ptr}(value, max_supply.read());

    // more code ...

    return ();
}

@external
func better_comparison{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    let (value: felt) = ERC20.total_supply();
    assert_nn_le{range_check_ptr=range_check_ptr}(value, max_supply.read());

    // more code ...

    return ();
}
```

**FIX**
- Review all comparisons closely to determine what sort of behaviour the comparison should have, and whether if assert_nn_le is more appropriate than assert_le.

## **Arithmetic Overflow**

**DESCRIPTION**
- The default primitive type, field element (felt), behvaes like an integer in many aspects but also has some important differnces.
- The range of valid felts is (-P/2, P/2), where P is the 252-bit prime number used by Cairo.
- Felt arithmetic is unchecked for underflow/overflow and this can lead to unexpected results.
- As the range of values spans both positive and negative, performing arithmetic on two positive numbers may result in negative value.
- This can also be seen in reverse, where multiplying two negative numbers may not result in a positve value.
- The Uint256 module is also unchecked, meaning that over/underflows is still possible.

```rust
@external
func overflow{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    num1: felt, num2: felt
) -> (res: felt) {
    // num1 (P-1) = 3618502788666131213697322783095070105623107215331596699973092056135872020480
    // num2 = 10
    return (num1 + num2);
    // Result = 9
}

@external
func addition_overflow_protection{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    num1: felt, num2: felt
) -> (res: felt) {
    sum = num1 + num2;
    with_attr error_message("Addition Overflow") {
        assert_le_felt(num1, res);
    }
    return (res=sum);
}

@external
func subtraction_underflow_protection{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}(num1: felt, num2: felt) -> (res: felt) {
    res = num1 - num2;
    with_attr error_message("Subtraction Underflow") {
        assert_le_felt(res, num1);
    }
    return (res);
}

@external
func multiplication_validation_protection{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}(num1: felt, num2: felt) -> (res: felt) {
    res = num1 * num2;
    with_attr error_message("Multiplication Overflow") {
        if (num2 != 0) {
            assert (res / num2) = num1;
        }
    }
    return (res);
}

@external
func multiplication_overflow_protection{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}(num1: felt, num2: felt) -> (res: felt) {
    res = num1 * num2;
    with_attr error_message("Multiplication Overflow") {
        if (num2 != 0) {
            assert (res / num2) = num1;
        }
    }
    return (res);
}
```

**FIX**
- Always add checks to ensure that the arithmetic is behaving as expected.
- Make use of various libraries, such as the OpenZeppelin Uint256 library or Nethermind Cairo SafeMath

## **Arithmetic Division**

**DESCRIPTION**
- Cairo math is performed over a finite field, hence the numeric type is called a felt (field element).
- Apart from overflows/underflows, addition, multiplication and subtraction will generally behave as expected in Cairo.
- In Cairo, it is more intuitive to think of division as the inverse of multiplication.
- When a number divides a whole number of times into another, the result is as it would be expected, i.e. 45/5=9
- If the numbers do not match up perfectly, the result can be unexpected i.e. 30/9=1206167596222043737899107594365023368541035738443865566657697352045290673497.
- This is because 120...97 * 9 = 30 (modulo the 252-bit Prime used by Cairo)

```rust
@external
func bad_scale_down_tokens{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    scaled_down_balance: felt
) {
    let (user) = get_caller_address();

    let (user_current_balance) = user_balances.read(user);
    let (scaled_down_balance) = user_current_balance / 10 ** 18;
    return (scaled_down_balance);
}

@external
func better_scale_down_tokens{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    ) -> (scaled_down_balance: Uint256) {
    let (user) = get_caller_address();

    let (user_current_balance) = user_balances.read(user);
    let (scaled_down_balance, _) = uint256_unsigned_div_rem(user_current_balance, 10 ** 18);
    return (scaled_down_balance);
}
```

**FIX**
- Review which numeric type is most appropriate for the use case.
- If the program relies on division, consider using the Starknet Uint256 module rather than the felt primitive.

## **Transfer of Important Variables**

**DESCRIPTION**
- When changing the values of important variables, such as ownership or admin variables, it is easy to accidentally input an incorrect value.
- There some be some form of check, which would ensure that any incorrect values won't immediately be live on the protocol.
- This could result in an owner or admin being set to a wrong address, and losing access to the protocol forever as it cannot be changed back.
- An example of this check would be timelocks, set then confirm or set then claim.

```rust
@storage_var
func owner() -> (res: felt) {
}

@storage_var
func proposed_owner() -> (res: felt) {
}

// INSECURE
func insecure_transfer_ownership{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    new_owner: felt
) {
    owner.write(new_owner);
    return ();
}

// MORE SECURE
func assert_only_owner{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    let (owner: felt) = owner.read();
    let (caller) = get_caller_address();

    with_attr error_message("Caller is not the owner") {
        assert_not_zero(caller);
        assert owner = caller;
    }
    return ();
}

func secure_transfer_ownership{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    new_owner: felt
) {
    assert_only_owner();

    with_attr error_message("New owner is zero address") {
        assert_not_zero(new_owner);
    }

    proposed_owner.write(new_owner);
    return ();
}

func secure_claim_ownership{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    let (caller) = get_caller_address();
    with_attr error_message("Caller is not proposed owner") {
        assert_not_zero(caller);
        assert proposed_owner = caller;
    }
    owner.write(proposed_owner);
    return ();
}

func secure_clear_proposed_ownership{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}() {
    assert_only_owner();
    proposed_owner.write(0);
    return ();
}
```

**FIX**
- There are various different safety methods to protect the change of important variables.
- The example above is for a set then claim, but you may also use set then confirm or even a timelock.
- Each of the safety features should contain the ability for the original owner to cancel the transfer at any point.

## **Multiple Contract Initializations**

**DESCRIPTION**
- When creating an upgradable contract, we want to make sure that the implementation contracts are protected.
- Implementation contracts should be initialized by contract developers and sensitive functions should have access control.
- The Proxy preset includes a fallback, an L1 handler and a constructor.
- Before deploying this Proxy contract, developers should declare the contract class of their implementation contract.
- After this, they can deploy the proxy contract and pass the implementation hash as a paramter.
- If somebody initializes the implementation contract before the developer and the developer doesn't notice, this could allow malicious actors to initialize certain contracts and would be set as the admin.

```rust
// INSECURE
@storage_var
func Proxy_implementation_hash() -> (class_hash: felt) {
}

@storage_var
func Proxy_admin() -> (proxy_admin: felt) {
}

func set_proxy{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    proxy_admin: felt, implementation_hash: felt
) {
    Proxy_admin.write(proxy_admin);
    Proxy_implementation_hash.write(implementation_hash);
    return ();
}

// MORE SECURE
@storage_var
func Proxy_initialized() -> (initialized: felt) {
}

@storage_var
func Proxy_implementation_hash() -> (class_hash: felt) {
}

@storage_var
func Proxy_admin() -> (proxy_admin: felt) {
}

func set_proxy{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    proxy_admin: felt, implementation_hash: felt
) {
    let (initialized) = Proxy_initialized.read();
    with_attr error_message("Contract already initialized"){
        assert initialized = FALSE
    }
    Proxy_initialized.write(TRUE);
    Proxy_implementation_hash.write(implementation_hash);
    Proxy_admin.write(proxy_admin);
    return ();
}
```

**FIX**
- Always make sure to use the OpenZeppelin initializable library for contracts with init functions, as well as ensuring the deployment also includes initialization.
- This will ensure that a contract cannot be initialized multiple times.
- The pattern should look like this: declare the implementation contracts first, then only deploy the Proxy, then finally, initialize the implementation contract via the Proxy.

## **Insecure State Modifications**

**DESCRIPTION**
- When using the @view decorator, ensure that no state is modified, as this can be very dangerous.
- Contrastingly, when using functions which modify state, ensure that the @external decorator is used.

```rust
@view
func bad_fetch_nonce{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    nonce: felt
) {
    let (user) = get_caller_address();
    let (nonce) = user_nonces.read(user);
    user_nonces.write(user, nonce + 1);
    return (nonce);
}

@view
func better_fetch_nonce{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    nonce: felt
) {
    let (user) = get_caller_address();
    let (nonce) = user_nonces.read(user);
    return (nonce);
}
```

**FIX**
- Ensure that view functions are only ever used to read data from storage or to perform calculations.
- As a best practice, it is recommended to review all function decorators inside the contracts and ensure that 'getters' have their visibility set as @view and 'setters' have their visibility set to @external.


## **Exposing Unwanted External Functions**

**DESCRIPTION**
- When importing modules with external functions, all will be automatically exposed. 
- Generally, this is something you might want, although it is important to ensure that sensitive functions are not exposed.

```rust
// library.cairo
%lang starknet
from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func owner() -> (res: felt){
}

func check_owner{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr: felt*}(){
    let caller = get_caller_address();
    let owner = owner.read();
    assert caller = owner;
    return ();
}

func do_something(){
    // do something potentially dangerous that only the owner can do
    return ();
}

@external
func bypass_owner_do_something(){
    do_something();
    return ();
}

// example.cairo
%lang starknet
%builtins pedersen range_check
from starkware.cairo.common.cairo_builtins import HashBuiltin
from library import check_owner(), do_something()
// Even though only check_owner() and do_something() are imported, we cam still call bypass_owner_do_something()
func check_owner_and_do_something{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr: felt*}(){
    bypass_owner_do_something();

    check_owner();
    do_something();
    return ();
}
```

**FIX**
- Exercise caution when declaring external functions in a library, and be aware that possible state changes that  can be made through the function and verify it is acceptable for anyone to call.
- To counter this, developers should follow the Extensibility pattern from OpenZeppelin.

## **Signature Replay Protection**

**DESCRIPTION**
- The StarkNet account abstraction model allows many authentication details to be offloaded to contracts.
- This provides greater flexibility, but signature schemas need to be created carefully to avoid replay attacks and signature malleability.
- To protect against this, it is suggested to use a unique nonce for each transfer.

```rust
// INSECURE

@external
func insecure_swap_currency{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    r: felt, s: felt, amount: felt
) {
    alloc_locals;
    let (local caller_address) = get_caller_address();
    let (_signer) = signer.read();
    let (message) = hash2{hash_ptr=pedersen_ptr}(amount, caller_address);

    verify_ecdsa_signature(message=message, public_key=_signer, signature_r=r, signature_s=s);

    let (_token_address) = token_address.read();
    let (contract_address) = get_contract_address();

    IERC20.transferFrom(
        contract_address=_token_address,
        sender=contract_address,
        recipient=caller_address,
        amount=Uint256(amount, 0),
    );

    return ();
}

// MORE SECURE

@storage_var
func nonces(address: felt) -> (nonce: felt) {
}

@external
func more_secure_swap_currency{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    r: felt, s: felt, amount: felt
) {
    alloc_locals;
    let (local caller_address) = get_caller_address();
    let (local nonce) = nonces.read(caller_address);
    let (_signer) = signer.read();

    // Update nonce
    nonces.write(caller_address, value=nonce + 1);

    let (message) = hash2{hash_ptr=pedersen_ptr}(amount, caller_address);
    let (message_part_2) = hash2{hash_ptr=pedersen_ptr}(message, nonce);

    verify_ecdsa_signature(
        message=message_part_2, public_key=_signer, signature_r=r, signature_s=s
    );

    let (_token_address) = token_address.read();
    let (contract_address) = get_contract_address();

    IERC20.transferFrom(
        contract_address=_token_address,
        sender=contract_address,
        recipient=caller_address,
        amount=Uint256(amount, 0),
    );

    return ();
}
```

**FIX**
- Signatures should include a nonce for each transfer via signature, which is stored inside a mapping and incremented each time it is used.
- This nonce is then used to generate the unique signature for each function call to avoid replay attacks.

## **Best Practices**

### **State is initialized to 0**
**DESCRIPTION**
- State will always be initialized to 0, which can lead to vulnerabilities.
- This can mean that checks where the value is set to 0 will automatically pass, even when it is not intended to.

### **Failing Asserts Without with_attr**
**DESCRIPTION**
- All asserts that can fail should have a with_attr to allow an error message that indicates the reason for the assert failing.
- This allows for easier debugging of the code.

### **Missing Use of Boolean Library**
**DESCRIPTION**
- If the values 0 and 1 are being used to represent boolean values, the boolean library should be used to imporve readability.

### **Missing Event Emission**
**DESCRIPTION**
- Events should be emitted for all important state changes, such as owner/admin addresses, token address and state initialization.

### **Testing Related Functions inside Core Contract**
**DESCRIPTION**
- Contracts being deployed to the mainnet should only contaun functions needed for the protocol to function.
- Testing related functions should be seperated from the main protocol logic.

### **Comments Not Reflecting Code**
**DESCRIPTION**
- Make sure that all comments within the code explain exactly what is actually happening in the code.

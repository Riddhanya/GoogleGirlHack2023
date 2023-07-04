# GoogleGirlHack2023
def read_circuit_file(file_path):
    circuit = {}
    with open(file_path, 'r') as file:
        for line in file:
            line = line.strip()
            if line:
                parts = line.split('=')
                gate = parts[0].strip()
                expression = parts[1].strip()
                circuit[gate] = expression
    return circuit

def evaluate_circuit(inputs, circuit):
    stack = []
    for gate, expression in circuit.items():
        operands = expression.split()
        if operands[0] == 'NOT':
            arg = stack.pop()
            result = not arg
        else:
            args = [stack.pop() for _ in range(len(operands))]
            args.reverse()
            if operands[1] == 'AND':
                result = all(args)
            elif operands[1] == 'OR':
                result = any(args)
            elif operands[1] == 'XOR':
                result = args[0] ^ args[1]
        stack.append(result)
    return stack.pop()

def generate_test_pattern(circuit_file, fault_node, fault_type):
    circuit = read_circuit_file(circuit_file)
    inputs = ['A', 'B', 'C', 'D']
    output = 'Z'
    
    # Set the fault value
    fault_value = 1 if fault_type == 'SA0' else 0
    
    # Determine the test pattern for the fault node
    test_pattern = {}
    test_pattern[fault_node] = fault_value
    
    # Evaluate the circuit to determine the remaining inputs
    for gate, expression in circuit.items():
        if gate != fault_node and gate != output:
            operands = expression.split()
            for operand in operands[2:]:
                if operand not in test_pattern:
                    test_pattern[operand] = 0
    
    # Convert the test pattern dictionary to a list
    test_vector = [test_pattern[input] for input in inputs]
    
    # Evaluate the circuit with the generated test pattern
    expected_output = evaluate_circuit(test_vector, circuit)
    
    return test_vector, expected_output

def write_output_to_file(output_file, inputs, expected_output):
    with open(output_file, 'w') as file:
        file.write(f"[{' '.join(inputs)}] = {inputs}, Z = {expected_output}\n")

# Example usage
circuit_file = "circuit.txt"
fault_node = "net_f"
fault_type = "SA0"
output_file = "output.txt"

test_vector, expected_output = generate_test_pattern(circuit_file, fault_node, fault_type)
write_output_to_file(output_file, test_vector, expected_output)

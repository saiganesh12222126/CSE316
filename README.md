#include <stdio.h>
#include <stdbool.h>

#define TOTAL_MEMORY 1000

struct MemoryBlock {
    int start;
    int end;
    int process_id;
    bool free;
};

struct MemoryBlock memory_blocks[1000];
int block_count = 1;

void initialize_memory() {
    memory_blocks[0].start = 0;
    memory_blocks[0].end = TOTAL_MEMORY;
    memory_blocks[0].process_id = -1;
    memory_blocks[0].free = true;
}

bool first_fit_allocation(int process_id, int required_memory) {
    int i, j;
    for (i = 0; i < block_count; i++) {
        if (memory_blocks[i].free && memory_blocks[i].end - memory_blocks[i].start >= required_memory) {
            memory_blocks[i].process_id = process_id;
            memory_blocks[i].free = false;
            if (memory_blocks[i].end - memory_blocks[i].start > required_memory) {
                for (j = block_count; j > i + 1; j--) {
                    memory_blocks[j] = memory_blocks[j - 1];
                }
                memory_blocks[i + 1].start = memory_blocks[i].start + required_memory;
                memory_blocks[i + 1].end = memory_blocks[i].end;
                memory_blocks[i].end = memory_blocks[i].start + required_memory;
                memory_blocks[i + 1].process_id = -1;
                memory_blocks[i + 1].free = true;
                block_count++;
            }
            return true;
        }
    }
    return false;
}

void deallocate_memory(int process_id) {
    int i;
    for (i = 0; i < block_count; i++) {
        if (memory_blocks[i].process_id == process_id) {
            memory_blocks[i].process_id = -1;
            memory_blocks[i].free = true;
        }
    }
}

int get_fragmentation() {
    int fragmentation = 0;
    int i;
    for (i = 0; i < block_count - 1; i++) {
        if (memory_blocks[i].free && memory_blocks[i + 1].free) {
            fragmentation += memory_blocks[i + 1].start - memory_blocks[i].end;
        }
    }
    return fragmentation;
}

int get_wasted_memory_blocks() {
    int wasted_blocks = 0;
    int i;
    for (i = 0; i < block_count; i++) {
        if (memory_blocks[i].free) {
            wasted_blocks++;
        }
    }
    return wasted_blocks;
}

void display_memory_status(int time_unit) {
    printf("Memory status at time unit %d:\n", time_unit);
    int i;
    for (i = 0; i < block_count; i++) {
        printf("Start: %d, End: %d, Process ID: %d, Free: %s\n",
               memory_blocks[i].start, memory_blocks[i].end, memory_blocks[i].process_id,
               memory_blocks[i].free ? "true" : "false");
    }
    printf("Fragmentation: %d\n", get_fragmentation());
    printf("Wasted memory blocks: %d\n", get_wasted_memory_blocks());
    printf("------------------------------------------------------------\n");
}

int main() {
    initialize_memory();
    int processes[][2] = {{1, 150}, {2, 200}, {3, 100}, {4, 300}};
    int num_processes = sizeof(processes) / sizeof(processes[0]);

    for (int time_unit = 1; time_unit <= num_processes; time_unit++) {
        int process_id = processes[time_unit - 1][0];
        int required_memory = processes[time_unit - 1][1];

        printf("Processing request at time unit %d - Process ID: %d, Required Memory: %d\n", time_unit, process_id, required_memory);
        if (first_fit_allocation(process_id, required_memory)) {
            printf("Memory allocated for Process ID %d\n", process_id);
        } else {
            printf("Memory allocation failed for Process ID %d\n", process_id);
        }
        display_memory_status(time_unit);
    }

    // Deallocate memory for completed processes
    deallocate_memory(2);
    display_memory_status(num_processes + 1);

    return 0;
}


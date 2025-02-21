#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

#define MAX_SHARES 100

// Function to convert a value from any base to decimal
unsigned long long convert_to_decimal(const char *value, int base) {
    unsigned long long decimal_value = 0;
    int len = strlen(value);
    for (int i = 0; i < len; i++) {
        char digit = value[i];
        int digit_value = (digit >= '0' && digit <= '9') ? digit - '0' : (digit - 'a' + 10);
        decimal_value += digit_value * pow(base, len - i - 1);
    }
    return decimal_value;
}

// Function for Lagrange Interpolation to find the secret (c)
unsigned long long lagrange_interpolation(int shares[][2], int n, int x_value) {
    double total = 0.0;
    for (int i = 0; i < n; i++) {
        double term = shares[i][1];  // yi
        for (int j = 0; j < n; j++) {
            if (i != j) {
                term *= (double)(x_value - shares[j][0]) / (shares[i][0] - shares[j][0]);
            }
        }
        total += term;
    }
    return (unsigned long long)total;
}

// Function to reconstruct the secret (constant term c)
unsigned long long reconstruct_secret(int n, int k, int shares[][2]) {
    return lagrange_interpolation(shares, k, 0);
}

// Function to read input from file
void read_input_file(const char *filename, int *n, int *k, int shares[][2]) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        printf("Error opening file.\n");
        exit(1);
    }

    char line[256];
    while (fgets(line, sizeof(line), file)) {
        // Parse the "keys" part (n and k values)
        if (strstr(line, "keys")) {
            sscanf(line, "keys: { \"n\": %d, \"k\": %d }", n, k);
        }
        // Parse each share
        else if (strchr(line, ':')) {
            int x;
            char base_str[10], value[256];
            sscanf(line, "%d: { \"base\": %s, \"value\": \"%s\" }", &x, base_str, value);
            
            int base = atoi(base_str);
            unsigned long long y = convert_to_decimal(value, base);
            shares[x - 1][0] = x;  // Set x value
            shares[x - 1][1] = y;  // Set corresponding y value
        }
    }

    fclose(file);
}

int main() {
    int shares[MAX_SHARES][2];
    int n, k;

    // Read input from the file
    read_input_file("input.txt", &n, &k, shares);

    printf("Reconstructing secret for test case with n = %d, k = %d...\n", n, k);
    unsigned long long secret = reconstruct_secret(n, k, shares);
    printf("Reconstructed secret: %llu\n", secret);

    return 0;
}

2nd test case:


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

unsigned long long convert_to_decimal(const char *value, int base) {
    unsigned long long decimal_value = 0;
    int len = strlen(value);
    for (int i = 0; i < len; i++) {
        char digit = value[i];
        int digit_value = (digit >= '0' && digit <= '9') ? digit - '0' : (digit - 'a' + 10);
        decimal_value += digit_value * pow(base, len - i - 1);
    }
    return decimal_value;
}

unsigned long long lagrange_interpolation(int shares[][2], int n, int x_value) {
    double total = 0.0;
    for (int i = 0; i < n; i++) {
        double term = shares[i][1];  // yi
        for (int j = 0; j < n; j++) {
            if (i != j) {
                term *= (double)(x_value - shares[j][0]) / (shares[i][0] - shares[j][0]);
            }
        }
        total += term;
    }
    return (unsigned long long)total;
}

unsigned long long reconstruct_secret(int n, int k, int shares[][2]) {
    return lagrange_interpolation(shares, k, 0);
}

int main() {
   
    int shares1[4][2] = {
        {1, convert_to_decimal("4", 10)},  // (1, 4)
        {2, convert_to_decimal("111", 2)}, // (2, 7)
        {3, convert_to_decimal("12", 10)}, // (3, 12)
        {6, convert_to_decimal("213", 4)}  // (6, 39)
    };

    unsigned long long secret1 = reconstruct_secret(4, 3, shares1);
    printf("secret key from test case 1: %llu\n", secret1);

   
    int shares2[10][2] = {
        {1, convert_to_decimal("420020006424065463", 7)},
        {2, convert_to_decimal("10511630252064643035", 7)},
        {3, convert_to_decimal("101010101001100101011100000001000111010010111101100100010", 2)},
        {4, convert_to_decimal("31261003022226126015", 8)},
        {5, convert_to_decimal("2564201006101516132035", 7)},
        {6, convert_to_decimal("a3c97ed550c69484", 15)},
        {7, convert_to_decimal("134b08c8739552a734", 13)},
        {8, convert_to_decimal("23600283241050447333", 10)},
        {9, convert_to_decimal("375870320616068547135", 9)},
        {10, convert_to_decimal("30140555423010311322515333", 6)}
    };

    unsigned long long secret2 = reconstruct_secret(10, 7, shares2);
    printf(" secret key from test case 2: %llu\n", secret2);

    return 0;
}

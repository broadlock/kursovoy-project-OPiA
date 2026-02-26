```
/******************************************************
 * Факультет: ФИТКБ
 * Группа: бИЦ - 251
 * ФИО: Гаркин Алексей Андреевич
 * Курсовая работа: Конструирование программы анализа функции
 ******************************************************/

#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <math.h>
#include <locale.h>

#ifdef _WIN32
#include <windows.h>
#endif

#define EPSILON 1e-10
#define STEP 0.001

double calculateFunctionValue(double x, int* status)
{
    *status = 1;

    if (x < -1)
    {
        return x * x * x - 2 * x + 5;
    }
    else if (x >= -1 && x < 1)
    {
        if (fabs(x) < EPSILON)
        {
            *status = 0;
            return 0;
        }
        return (exp(x) - exp(-x)) / (2 * x);
    }
    else
    {
        double result = 1.0;
        result -= (x * x) / 2.0;
        result += pow(x, 4) / 24.0;
        result -= pow(x, 6) / 720.0;
        result += pow(x, 8) / 40320.0;
        return result;
    }
}

double calculateDerivativeAnalytic(double x, int* status)
{
    *status = 1;

    if (x < -1)
    {
        return 3 * x * x - 2;
    }
    else if (x > -1 && x < 1)
    {
        if (fabs(x) < EPSILON)
        {
            *status = 0;
            return 0;
        }
        double numerator = exp(x) + exp(-x);
        double denominator = 2 * x;
        return (numerator * denominator - (exp(x) - exp(-x)) * 2) / (denominator * denominator);
    }
    else if (x > 1)
    {
        double result = 0.0;
        result -= x;
        result += pow(x, 3) / 6.0;
        result -= pow(x, 5) / 120.0;
        result += pow(x, 7) / 5040.0;
        return result;
    }
    else
    {
        *status = 1;
        double h = 1e-7;
        int status1 = 1, status2 = 1;
        double fx_plus = calculateFunctionValue(x + h, &status1);
        double fx_minus = calculateFunctionValue(x - h, &status2);
        if (status1 == 0 || status2 == 0)
        {
            *status = 0;
            return 0;
        }
        return (fx_plus - fx_minus) / (2 * h);
    }
}

void findMinMax(double a, double b)
{
    int status;
    double min_val, max_val;
    double min_x = a, max_x = a;
    int first = 1;

    printf("\nПоиск экстремумов на отрезке [%.3f; %.3f]\n", a, b);
    printf("------------------------------------------------\n");

    for (double x = a; x <= b + EPSILON; x += STEP)
    {
        double fx = calculateFunctionValue(x, &status);

        if (status == 0) continue;

        if (first)
        {
            min_val = max_val = fx;
            min_x = max_x = x;
            first = 0;
        }
        else
        {
            if (fx < min_val)
            {
                min_val = fx;
                min_x = x;
            }
            if (fx > max_val)
            {
                max_val = fx;
                max_x = x;
            }
        }
    }

    if (!first)
    {
        printf("Минимум: f(%.4f) = %.6f\n", min_x, min_val);
        printf("Максимум: f(%.4f) = %.6f\n", max_x, max_val);
    }
    else
    {
        printf("Не удалось найти значения функции на отрезке\n");
    }
}

void checkMonotonicity(double a, double b)
{
    int status;
    int increasing = 1;
    int decreasing = 1;
    double prev_x = a;
    double prev_fx = calculateFunctionValue(prev_x, &status);

    printf("\nПроверка на монотонность на отрезке [%.3f; %.3f]\n", a, b);
    printf("------------------------------------------------\n");

    if (status == 0)
    {
        printf("Функция не определена в начальной точке\n");
        return;
    }

    for (double x = a + STEP; x <= b + EPSILON; x += STEP)
    {
        double fx = calculateFunctionValue(x, &status);

        if (status == 0) continue;

        if (fx < prev_fx - EPSILON)
        {
            increasing = 0;
        }
        if (fx > prev_fx + EPSILON)
        {
            decreasing = 0;
        }

        prev_x = x;
        prev_fx = fx;
    }

    if (increasing)
    {
        printf("Функция возрастает на отрезке\n");
    }
    else if (decreasing)
    {
        printf("Функция убывает на отрезке\n");
    }
    else
    {
        printf("Функция не является монотонной на отрезке\n");
    }
}

void printTable(double a, double b, double step)
{
    int status;

    printf("\nТаблица значений функции на отрезке [%.3f; %.3f]\n", a, b);
    printf("%-15s | %-15s\n", "x", "f(x)");
    printf("-----------------|-----------------\n");

    for (double x = a; x <= b + EPSILON; x += step)
    {
        double fx = calculateFunctionValue(x, &status);

        if (status == 0)
        {
            printf("%-15.4lf | %s\n", x, "Не определено");
        }
        else
        {
            printf("%-15.4lf | %-15.6lf\n", x, fx);
        }
    }
}

void showMenu()
{
    printf("\n========================================\n");
    printf("           МЕНЮ ПРОГРАММЫ\n");
    printf("========================================\n");
    printf("1. Вычислить значение f(x) в точке\n");
    printf("2. Таблица значений x -> f(x) на интервале\n");
    printf("3. Найти min/max (экстремумы) на отрезке\n");
    printf("4. Проверка на монотонность на интервале\n");
    printf("5. Вычислить производную f'(x) в точке\n");
    printf("0. Выход\n");
    printf("========================================\n");
    printf("Выберите пункт меню: ");
}

void initializeConsole()
{
#ifdef _WIN32
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);
    setlocale(LC_ALL, "Russian");
#else
    setlocale(LC_ALL, "ru_RU.UTF-8");
#endif
}

int main()
{
    initializeConsole();

    printf("******************************************************\n");
    printf("* Факультет: ФИТКБ                                  *\n");
    printf("* Группа: бИЦ - 251                                 *\n");
    printf("* ФИО: Гаркин Алексей Андреевич                     *\n");
    printf("* Курсовая работа: Конструирование программы        *\n");
    printf("*         анализа функции                           *\n");
    printf("******************************************************\n");

    int choice;
    double x, a, b, step;
    int status;
    double fx, fprime;

    do
    {
        showMenu();

        if (scanf("%d", &choice) != 1)
        {
            printf("Ошибка ввода. Попробуйте снова.\n");
            while (getchar() != '\n');
            continue;
        }

        switch (choice)
        {
        case 1:
            printf("\nВведите значение x: ");
            if (scanf("%lf", &x) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            fx = calculateFunctionValue(x, &status);

            if (status == 0)
            {
                printf("Функция не определена для x = %.5lf\n", x);
            }
            else
            {
                printf("f(%.5lf) = %.6lf\n", x, fx);
            }
            break;

        case 2:
            printf("\nВведите начало интервала a: ");
            if (scanf("%lf", &a) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            printf("Введите конец интервала b: ");
            if (scanf("%lf", &b) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            printf("Введите шаг таблицы: ");
            if (scanf("%lf", &step) != 1 || step <= 0)
            {
                printf("Ошибка ввода шага.\n");
                break;
            }

            if (a > b)
            {
                double temp = a;
                a = b;
                b = temp;
            }

            printTable(a, b, step);
            break;

        case 3:
            printf("\nВведите начало отрезка a: ");
            if (scanf("%lf", &a) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            printf("Введите конец отрезка b: ");
            if (scanf("%lf", &b) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            if (a > b)
            {
                double temp = a;
                a = b;
                b = temp;
            }

            findMinMax(a, b);
            break;

        case 4:
            printf("\nВведите начало интервала a: ");
            if (scanf("%lf", &a) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            printf("Введите конец интервала b: ");
            if (scanf("%lf", &b) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            if (a > b)
            {
                double temp = a;
                a = b;
                b = temp;
            }

            checkMonotonicity(a, b);
            break;

        case 5:
            printf("\nВведите значение x: ");
            if (scanf("%lf", &x) != 1)
            {
                printf("Ошибка ввода.\n");
                break;
            }

            fprime = calculateDerivativeAnalytic(x, &status);

            if (status == 0)
            {
                printf("Производная не определена для x = %.5lf\n", x);
            }
            else
            {
                printf("f'(%.5lf) = %.6lf\n", x, fprime);
            }
            break;

        case 0:
            printf("\nВыход из программы.\n");
            break;

        default:
            printf("\nНеверный пункт меню. Попробуйте снова.\n");
        }

    } while (choice != 0);

    return 0;
}
```

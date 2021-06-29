/** @file Практика.cpp
* @author Федоров Р.С., гр. 515б,вариант 33
* @date 27.06.21
* @brief Практическая робота
*/




#define _CRT_SECURE_NO_WARNINGS
#include <iostream>
#include <windows.h>
#include <string>
#include <iomanip>

#include <random>
using namespace std;

/*
	Розробити програму, що переводить суму, задану у вигляді числа з плаваючою
	комою, в текст українською мовою. Приклади:

	23.08 -> двадцять три гривні вісім копійок
	11.50 -> одинадцять гривень п'ятдесят копійок
	0.30  -> тридцять копійок

*/


// випадкове число
int randInt(int min, int max)
{
	int r = rand() % (max - min + 1) + min;
	return r;
}

// генерує випадкове число довжиною в n цифр в форматі грошей
// у вигляді строки
string RandomNumer(int n)
{
	string v;
	for (int i = 0; i < n; i++) {
		char digit = randInt(0, 9) + 0x30;
		v += digit;
	}
	int len = v.size();
	const char* dt = v.c_str();
	if (dt[len - 1] == 0x30)v.erase(len - 1, 1); // остання цифра ==0
	if (dt[len - 1] == 0x30)v.erase(len - 1, 1); // передостання цифра ==0
	if (v.size() > 2 && v.size() > len - 2)v.insert(len - 2, 1, '.');
	if (v.size() == 1 || v.size() == 2)v = "0." + v;
	return v;
}



string decF2[]{ "нуль",  "один", "два", "три", "чотири", "п'ять", "шість", "сім", "вісім", "дев'ять", "десять" };
string decF1[]{ "нуль", "одна", "дві", "три", "чотири", "п'ять", "шість", "сім", "вісім", "дев'ять", "десять" };
string* decF;


string tw[]{ "десять","одинадцять","дванадцять","тринадцять","чотирнадцять","п'ятнадцять",
			"шістнадцять","сімнадцять","вісімнадцять","дев'ятнадцять","двадцять" };

string ten[]{ "нуль","десять", "двадцять","тридцять","сорок","п'ятдесят","шістдесят",
			"сімдесят","вісімдесят","дев'яносто","сто" };

string hnd[]{ "нуль","сто","двісті","триста","чотириста","п'ятсот","шістсот",
"сімсот","вісімсот","дев'ятсот","тисяча", };

string Ntau[]{ "тисяч","тисяча","тисячі","тисячі","тисячі","тисяч","тисяч","тисяч","тисяч","тисяч","тисяч" };

string Nmil[]{ "мільйонів","мільйон","мільйони","мільйони","мільйони","мільйонів","мільйонів","мільйонів","мільйонів","мільйонів","мільйонів" };
string Ngr[]{ "гривень","гривня","гривні","гривні","гривні","гривень","гривень","гривень","гривень","гривень","гривень" };
string Nkop[]{ "копійок", "копійка","копійки","копійки","копійки","копійок","копійок","копійок","копійок","копійок","копійок" };



bool is_digit(string s)
{
	const char* d = s.c_str();
	for (int i = 0; i < s.size(); i++) {
		if (d[i] < 0x30 || d[i]>0x39)return false;
	}
	return true;
}


string DigToWords(string v, int f = 0)
{
	if (f)decF = decF2; else decF = decF1;

	int size = v.size();
	if (size == 0)return "";
	const char* dt = v.c_str();

	if (size == 1)return decF[dt[0] - 0x30];

	if (size == 2) {
		int x = stoi(v);
		if ((x % 10) == 0)return ten[x / 10];
		else {
			int x = stoi(v);
			if (x > 9 && x < 20)return tw[x - 10];
			else {
				if (dt[0] == 0x30)return decF[dt[1] - 0x30];
				return ten[dt[0] - 0x30] + " " + decF[dt[1] - 0x30];
			}
		}
	}

	if (size == 3) { // сотні
		int x = stoi(v);
		if ((x % 100) == 0)return hnd[x / 100];
		else {
			string xx = v.erase(0, 1);
			if (x < 100)return DigToWords(xx);
			else return hnd[x / 100] + " " + DigToWords(xx, f);
		}
	}

	if (size > 3 && size < 7) { // тисячі
		string st = v.substr(0, size - 3);
		string ml = v.substr(size - 3, 3);
		const char* st_dt = st.c_str();
		int LastDigit = st_dt[st.size() - 1] - 0x30;

		if (stoi(v) < 1000)return DigToWords(ml, f); //000123

		string t_name = Ntau[LastDigit];
		string s_xx = st; if (s_xx.size() == 3)s_xx.erase(0, 1);
		int xx = stoi(s_xx); // дві останні цифри
		if (xx > 9 && xx < 21)t_name = "тисяч";

		if (stoi(ml) == 0) { return DigToWords(st, f) + " " + t_name; } // 123000
		else return DigToWords(st, 0) + " " + t_name + " " + DigToWords(ml, f);

	}

	if (size > 6 && size < 10) { // мільйони
		string mil = v.substr(0, size - 6);
		string ts = v.substr(size - 6, 3);
		string od = v.substr(size - 3, 3);
		const char* mil_dt = mil.c_str();
		int LastDigit = mil_dt[mil.size() - 1] - 0x30;

		if (stoi(v) < 1000)return DigToWords(od); // типу 000000123
		if (stoi(v) < 1000000)return DigToWords(ts + od); // типу 000123456
		if (stoi(ts) == 0) {//типу 123000456 #
			string xx = mil; if (mil.size() == 3)xx.erase(0, 1);
			int m = stoi(xx);
			if (m > 9 && m < 20)return DigToWords(mil, 1) + " мільйонів " + DigToWords(od);
			return DigToWords(mil, 1) + " " + Nmil[LastDigit] + " " + DigToWords(od); //типу 123000456 #
		}

		if (stoi(ts + od) == 0) { return DigToWords(mil, 1) + " " + Nmil[LastDigit]; } // 123000000
		else {
			string xx = mil; if (mil.size() == 3)xx.erase(0, 1);
			int m = stoi(xx);
			if (m > 9 && m < 20)return DigToWords(mil, 1) + " мільйонів " + DigToWords(ts + od);
			return DigToWords(mil, 1) + " " + Nmil[LastDigit] + " " + DigToWords(ts + od);
		}
	}

}

void PrintManyString(string v)
{
	// розбиваємо строку на гривні і копійки
	string kop, grn;
	int pos = v.find('.');
	if (pos == v.npos) { grn = v; } // копійок немає
	else {
		grn = v.substr(0, pos); // cout << grn << endl;
		string tmp = v;
		kop = tmp.erase(0, pos + 1); // cout << kop << endl;
		if (kop.size() == 1)kop += "0"; // 0.7 ---> 70, а не 7
	}
	if (!is_digit(grn)) { cout << " wrong data! \n"; return; }
	if (!is_digit(kop)) { cout << " wrong data! \n"; return; }

	string name_gr;
	if (stoi(grn) < 10)name_gr = Ngr[stoi(grn)];
	else {
		string xx_g = grn; if (xx_g.size() == 3)xx_g.erase(0, 1);
		int xx = stoi(xx_g);
		if (xx > 9 && xx < 21)name_gr = "гривнів";
		else name_gr = Ngr[xx % 10];
	}

	int xx = 0;
	if (kop.size())xx = stoi(kop);
	string name_kop = Nkop[xx % 10];
	if (xx > 9 && xx < 21)name_kop = "копійок";

	if (kop.size()) {
		if (stoi(grn) == 0)cout << v << " -> " << DigToWords(kop) << " " + name_kop << " " << endl;
		else
			cout << v << " -> " << DigToWords(grn) << " " + name_gr << " " << DigToWords(kop) << " " + name_kop << " " << endl;
	}
	else cout << v << " -> " << DigToWords(grn) << " " + name_gr << endl;
}

int main()
{
	SetConsoleOutputCP(1251); 
	SetConsoleCP(1251);  

	std::default_random_engine gen;
	std::uniform_int_distribution<int> distribution(1, 999999999);



	cout << "Random generation numers:\n";



	for (int i = 0; i < 30; i++) {
		string grn = to_string(distribution(gen));
		int len = grn.size();
		grn = grn.substr(0, randInt(1, len - 1));
		string many = grn + "." + to_string(randInt(0, 99));
		PrintManyString(many);
	}

	PrintManyString("7");
	PrintManyString("0.7");
	PrintManyString("0.07");
	PrintManyString("0.79");

	// ручний ввід числа
	cout << "\nenter data (max 11 digits):\n";
	string v; cin >> v;
	if (v.size() > 12)cout << "Дуже довге число!\n";
	else PrintManyString(v);




	cout << "\n\n"; system("pause"); 
	return 0;
}

# PAS-24
Proyek Akhir Semester 2 Kelompok 24. U-Fit merupakan program tracking target harian dengan parameter berupa laporan diet harian dan target diet pengguna. Dilengkapi dengan fitur penghitung Indeks Massa Tubuh serta identifikasi kategori berat badan untuk menjadi konsiderasi pengguna dalam memakai aplikasi dan menjalani diet sehatnya.
-----------------------------------------------------------------------------------------------------
/*A Diet Tracker program made by Kelompok 24
"Set and Track your nutrition to get your target. Healtier body, Happier you"

Azzah Azkiyah Angeli Syahwa - 2106731390
Muhammad Irsyad Fakhruddin - 2006468850	*/

#include <stdio.h> //library for input output operation
#include <stdlib.h> //library for dynamic memory alocation, variables, etc
#include <omp.h> //library for accessing openMP
#include <windows.h> //declarations for all of the functions in the Windows API

struct diet{ //contains variables for "Laporan harian" will be stored at "userDiet.txt"
	int BB;
	int Carbs;
	int Fat;
	int Protein;
	int Olahraga;
	int Air;
	
	struct diet *next;
};

typedef struct User{ //contains variables for "Atur/Ubah Target" option. will be stored at "Target.txt"
	char nama[30];
	int umur;
	int target_status;
	int tBB;
	int tCarbs;
	int tFat;
	int tProtein;
	int tOlahraga;
	int tAir;
	int hari;
	
	struct diet *data;
}User;

typedef struct diet Diet;
typedef Diet *Dietptr;

struct Nutrisi{
    char sumberCar[30];
    char sumberPro[30];
    char sumberFat[30];
    int sumkcalCar;
    int sumkcalPro;
    int sumkcalFat;
    int car;
    int pro;
    int fat;
    int kcal;
    
    struct Nutrisi *next_ntr;
};

//prototypes menu
void welcome(User *userptr);
void countBMI();
void countKalori();
void menu_target();
void input_data(Dietptr *sptr);
void printDiet(Dietptr current, int day_removed[50], User *userptr);
void print_evaluasi(Dietptr current, User *userptr , int day_removed[50]);
void removeptr (Dietptr *startPtr, int day, int day_removed[50]);
void countPeriode();
void information();
void help();
//prototypes file handling
int file_user_read (User *userptr, int day_removed[50]);
int file_user_write (User *userptr, int day_removed[50]);
int file_target_write (Dietptr current, int day_removed[50]);
int file_target_read (Dietptr *sptr, int day_removed[50], int i, int *posisi);
int file_removed_write(int day[50]);
int file_removed_read(int day[50]);


int main(){
	system("color e0"); //set background color "Yellow" and "Black" colored font.
	FILE *fptr;
	
	int menu, hari, i, j, id, target, status, login, login_status, file_status, posisi, pilihan;
	int sumCarbs, sumProtein, sumFat, sumKal;
	int day_removed[50] = {}; //an array for Laporan harian's removed day
	char temp; //variable for handling the space on a name 
	
	//set menu, target, and id at 0 to avoid "garbage value"
	menu = 0;
	target = 0;
	id = 0;
	
	User *userptr; //declaration to call User struct using pointer
	
	userptr = (User*) calloc(1, 2 * sizeof(User)); //pointer struct for single-user
	
	Dietptr startptr = NULL; //initialize start pointer as NULL
	
	//reading file
	file_status = file_user_read(userptr, day_removed); //file data user
	file_removed_read(day_removed); //file removed day
	
	//set Posisi to read files at 0 to avoid "garbage value"
	posisi = 0;
	
	//if file_target_read hasn't been created 
	if((userptr + id)->hari == 0){
		file_target_read(&startptr, day_removed, 0, &posisi);
	}
	//if file_target_read has been created 
	else{
	//the loop is executed as many days as the data entered by the user
		for(i = 0; i < (userptr + id)->hari - 1; i++){
			file_target_read(&startptr, day_removed, i, &posisi);
		}
		//last loop passes value -1 to close the file
		file_target_read(&startptr, day_removed, -1, &posisi);
	}
	//link from userptr is linked to the starting address of link list Diet (startptr)
	(userptr + id)->data = startptr; 
	
	while (menu != -1){
		welcome(userptr);
		login_status = 1; //initialize login_status as 1 
		printf("\nMasukan angka: ");
		scanf("%d", &menu);
		system("cls");
		
		target = 0; //initialize target as 0 
		switch (menu){
			case 1:
				countBMI(); //option 1 : Menghitung BMI/IMT, function to count body mass indeks and other infos
				printf("\n\n");
				system("pause");
				system("cls");
				break;
			case 2: 
			//option 2 : Atur atau ubah Target, function to set/change target, evaluate, and remove the daily reports.
				while(target != -1){ //do the looping as long as target is not -1
					menu_target();
					printf("\nMasukan pilihan : ");
					scanf("%d", &target);
					system("cls");
					switch(target){
					case 1: //set or change target
						printf("\nNama : ");
						scanf("%c", &temp);
						scanf("%[^\n]", &(userptr + id)->nama);
						printf("\nUmur: ");
						scanf("%d", &(userptr + id)->umur);
						if((userptr + id)->umur < 19){ //error and warning handling. underaged users are not recommended to do diet cutting.
							printf("Umur anda masih dalam umur perkembangan.");
							printf("\nSehingga, melakukan diet atau membatasi asupan makan tidak baik dilakukan pada saat ini.");
							system("pause");
							system("cls");
						}
						else{	//if user's age is pass 19, they're allowed.
							(userptr + id)->target_status = 1; //start for input the user's identity and diet target
							printf("Masukan Target Berat Badan \t: ");
							scanf("%d", &(userptr + id)->tBB);
							printf("Masukan Target Karbohidrat \t: ");
							scanf("%d", &(userptr + id)->tCarbs);
							sumCarbs = (userptr + id)->tCarbs * 4;
							printf("Masukkan Target Protein \t: ");
							scanf("%d", &(userptr + id)->tProtein);
							sumProtein = (userptr + id)->tProtein * 4;
							printf("Masukkan Target Lemak \t\t: ");
							scanf("%d", &(userptr + id)->tFat);
							sumFat = (userptr + id)->tFat * 9;
							sumKal = (sumCarbs+sumProtein+sumFat);
							printf("\nJumlah target kalori anda adalah %d kcal", sumKal);
							if(sumKal >= 1500){ //if condition. to avoid unhealty diet cutting 
								printf("\nTarget kalori anda terlalu besar, saat melakukan diet cutting");
								printf("\nkami menyarankan 1000-1500 kcal perhari.");
							}
							else{
								(userptr + id)->tAir = 2; //set the tAir as 2 liters (min amount of water human needed)
								printf("\nMasukan Target Olahraga [mohon tuliskan dalam menit, contoh : 90] \t: ");
								scanf("%d", &(userptr + id)->tOlahraga);
								printf("\n\n");
								system("pause");
								system("cls");
							}
							}	
						break;	
					case 2: //input daily report
						if((userptr + id)->target_status != 1){ //user should set the targets first to continue.
							printf("\nHarap masukan target terlebih dahulu\n\n");
							system("pause");
							system("cls");
							break;
						}					
						printf("\n\t=====================================");
						printf("\n\t==                                 ==");
						printf("\n\t==         Laporan Harian          ==");
						printf("\n\t==                                 ==");		
						printf("\n\t=====================================");
						printf("\nHalo, %s!", (userptr + id)->nama);
						printf("\nMasukan jumlah hari yang akan diinput: ");
						scanf("%d", &hari);
						for(i = 0; i < hari; i++){ 	//the loop is executed as many days as the data entered by the user
							printf("\n\nDaily Report hari ke-%d", (userptr->hari) + i + 1);
							//input_data is passed the address of the start pointer as the start pointer in the listed link
							input_data(&startptr);
						}
						//struct of userptr linked to starting pointer link listed Laporan Harian
						(userptr + id)->data = startptr; 
						system("cls");
						printf("\n\nInput berhasil!\n");
						printDiet((userptr + id)->data, day_removed, userptr);
						printf("\n\n");
						system("pause");
						system("cls");
						break;
					case 3: //evaluate the targets
						if((userptr + id)->target_status != 1){ //user should set the targets first to continue.
							printf("\nHarap masukan target terlebih dahulu\n\n");
							system("pause");
							system("cls");
							break;
						}
						printf("\nHalo, %s!", (userptr + id)->nama);
						printf("\nUmur mu saat ini %d tahun", (userptr + id)->umur);
						printDiet((userptr + id)->data, day_removed, userptr);
						printf("\n\n");
						system("pause");
						system("cls");
						print_evaluasi((userptr + id)->data, userptr, day_removed);
						printf("\n\n");
						system("pause");
						system("cls"); 
						break;
					case 4: //remove a day or more on user's daily report
						if((userptr + id)->target_status != 1){ //user should set the targets first to continue.
							printf("\nHarap masukan target terlebih dahulu\n\n");
							system("pause");
							system("cls");
							break;
						}
						printf("Pilih hari yang akan dihapus: ");
						scanf("%d", &hari);
						hari--;
						if(hari >= (userptr + id)->hari){
							printf("\n\nDaily Report di hari ke %d belum pernah diinput!\n\n", hari+1);
							system("pause");
							system("cls");
							break;
						}
						removeptr (&startptr, hari, day_removed);
						day_removed[hari] = 1;
						printf("\nData hari ke-%d berhasil dihapus\n\n", hari+1);
						system("pause");
						system("cls");
						break;     
				}
			}
			break;
			case 3:	
				countPeriode(); //option 3: Menghitung Periode Target. function to calculates weeks needed to reach the target weight
				printf("\n\n");
				system("pause");
				system("cls");
				break;	
			case 4:
				information(); //option 4 : Informasi Seputar Diet. function that contains informations about Diet
				printf("\n\n");
				system("pause");
				system("cls");
				break;
			case 5:
        		help(); //option 5 : Bantuan. function that contains information about how to use the program
        		printf("\n\n");
				system("pause");
				system("cls");	
				break;	
		}
	}	
	//call the file handling functions 
	file_user_write(userptr, day_removed);
	file_target_write((userptr + id)->data, day_removed);
	file_removed_write(day_removed);
	
	return 0;
}

void welcome(User *userptr){ //a starting point of the app, displaying U-Fit's menu
	int i, tid;
	
	#pragma omp for //parallel using for
	for(i = 0; i < 30; i++){
		printf("=*");
	}
	printf("\n             \t\t\t\n");
	printf("\n\t\t\t U-Fit\n");
	printf("\n             \t\t\t\n");

	#pragma omp for //multithreads to save time
	for(i = 0; i < 30 ;i++){
		printf("=*");
	}

	//Menu of 5 Access Options		
	printf("\n\nMenu U-Fit: ");	
	printf("\n1. Menghitung BMI/IMT");
	printf("\n2. Atur/Ubah Target");
	printf("\n3. Menghitung Periode Target");
	printf("\n4. Informasi Seputar Diet" );
 	printf("\n5. Bantuan");
	printf("\n-1. Keluar\n\n");
}

void countBMI(){ 
//calculate user's BMI as a consideration to do the diet, and giving 
// such a information ideal and healthy weight and user's daily basal calorie.
	float tinggi, berat, KKB = 0; 
	int input = 0;
	
	printf("\n\t\t=====================================");
	printf("\n\t\t==                                 ==");
	printf("\n\t\t==        Indeks Masa Tubuh        ==");
	printf("\n\t\t==        (Body Mass Index)        ==");	
	printf("\n\t\t==                                 ==");	
	printf("\n\t\t=====================================");
	
	printf("\nMengetahui Indeks Massa Tubuh atau Body Mass Index penting untuk dilakukan");
	printf("\nsebelum kamu memutuskan untuk melakukan diet cutting.");	
	printf("\n\n<> Gender : \n(1)Wanita\n(2)Pria\n= ");
	scanf("%d", &input);
	printf("\n\n<> Masukkan tinggi badan saat ini : ");
	scanf("%f", &tinggi);
	printf("\n<> Masukkan berat badan saat ini : ");
	scanf("%f", &berat);
	
	float tKuadrat = (tinggi*tinggi)/10000; 
	float uBMI = berat/tKuadrat; //BMI's calculation formula, retrieved from Ministry of Health website.
	
	//if condition to categorize user's BMI
	if(uBMI <= 18.5){
		printf("\n\n===========> BMI Result");
		printf("\nBerdasarkan perhitungan, BMI mu berada di angka %f", uBMI);
		printf("\ndan masuk pada kategori underweight.");
		printf("\nkamu sangat TIDAK DISARANKAN melakukan diet cutting");
		printf("\nsilahkan melakukan diet bulking.");
	}
	else if(18.5 < uBMI <= 22.9){
		printf("\n\n===========> BMI Result");	
		printf("\nBerdasarkan perhitungan, BMI mu berada di angka %f", uBMI);
		printf("\ndan masuk pada kategori Normal.");
	}
	else if(23 < uBMI <= 23.9){
		printf("\n===========> BMI Result");
		printf("Berdasarkan perhitungan, BMI mu berada di angka %f", uBMI);
		printf("\ndan masuk pada kategori overweight beresiko.");
		printf("\nkamu DISARANKAN melakukan DIET CUTTING.");
	}
	else if(uBMI >= 25){
		printf("\n===========> BMI Result");
		printf("Berdasarkan perhitungan, BMI mu berada di angka %f", uBMI);
		printf("\ndan masuk pada kategori Obesitas.");
		printf("\n[!]Obesitas memiliki tingkatan yang masing-masing beresiko");
		printf("\nkami SANGAT MENYARANKAN kamu untuk ditangani oleh profesional.");
	}
	
	float sub = tinggi-100;
	float BBideal = sub-(0.1*sub); //Ideal Weight's calculation formula, retrieved from Ministry of Health website.
	
	printf("\n\n===========> Informasi Tambahan");
	printf("\n<> Berat idealmu : %f kg", BBideal);
	if(input == 1){
		KKB = 25*BBideal; //Basal Calorie's for women calculation formula, retrieved from Ministry of Health website.
		printf("\n<> Kebutuhan kalori basal (kalori harian) kamu adalah %f kcal", KKB); 
	}
	else if(input == 2){
		KKB = 30*BBideal; //Basal Calorie's for men calculation formula, retrieved from Ministry of Health website.
		printf("\n<> Kebutuhan kalori basal (kalori harian) kamu adalah %f", KKB);
	}	
}

void menu_target(){ 
//displaying options for 1. Atur/Ubah Target

	printf("\n1. Input / Ubah Target Diet");
	printf("\n2. Input Pelaksanaan Harian");
	printf("\n3. Lihat Evaluasi Target");
	printf("\n4. Menghapus Target Harian");
	printf("\n-1. Exit");
}

void input_data(Dietptr *sptr){
//user input func

	Dietptr currentptr; //pointer to current address
	Dietptr newptr; //new pointer to insert
	Dietptr prevptr; //previous pointer before currentptr
	
	newptr = malloc(sizeof(Diet));
		
	//user input for daily report
	printf("\nA. Berat Badan saat ini? \n ");
	scanf("%d", &newptr->BB);
	printf("B. Hari ini sudah konsumsi berapa karbohidrat?: \n");
	scanf("%d", &newptr->Carbs);
	printf("C. Hari ini sudah konsumsi berapa protein?: \n");
	scanf("%d", &newptr->Protein);
	printf("D. Hari ini sudah konsumsi berapa lemak?: \n");
	scanf("%d", &newptr->Fat);
	printf("E. Olahraga hari ini?: \n");
	scanf("%d", &newptr->Olahraga);
	printf("F. Sudah mengonsumsi berapa banyak air?: \n");
	scanf("%d", &newptr->Air);

	//initialize nextptr and prevptr as NULL
	newptr->next = NULL;
	prevptr = NULL;
	//now currentptr has the same address as the starter pointer
	currentptr = *sptr;
	
	//while loop until the end of pointer (currentptr found a NULL)
	while(currentptr != NULL){
		prevptr = currentptr;
		currentptr = currentptr->next;
	}
	//if the list is NULL, do the instruction
	if (prevptr == NULL){
		//insert newptr at the beginning of the linked list
		newptr->next = *sptr;
		*sptr = newptr;
	}
	//if the link list is filled then newptr will be inserted
	else{    
		prevptr->next = newptr;
		newptr->next = currentptr;
	}	
}

void printDiet(Dietptr current, int day_removed[50], User *userptr){
	
	int counter;
	counter = 0; //set counter at 0 to avoid garbage value
	
	while(current != NULL){
		if(day_removed[counter] == 1){
			printf("\n\nLaporan Harian dietmu di hari ke-%d telah dihapus \n", counter + 1);
		}
		else{
		
		#pragma omp parallel //parallel programming implementation
		{
			int tid;
		
			tid = omp_get_thread_num();
			
			#pragma omp single //using a single thread
			{
				printf("\n\nRekap Diet hari ke-%d:", counter + 1);	
			}
			
			#pragma omp taskwait //parallel taskwait to avoid race condition of the thread
			
			//task distribution 
			if(tid == 0){ 
				printf("\nBerat badanmu saat ini %d kg",current->BB);
				printf("\nKarbohidrat yang kamu konsumsi %d gram",current->Carbs);
			}
			if (tid == 1){
				printf("\nProtein yang kamu konsumsi %d gram",current->Protein);
				printf("\nLemak yang kamu konsumsi %d gram",current->Fat);
			}
			if(tid == 2){
				printf("\nKamu hari ini berolahraga selama : %d jam",current->Olahraga);
				printf("\nKamu  sudah mememinum %d liter air",current->Air);
			}
			
			#pragma omp taskwait 	//parallel taskwait to avoid race condition of the thread
		}
	}
		current = current->next;
		counter++;
	}
	
	userptr->hari = counter;
		
}

void print_evaluasi(Dietptr current, User *userptr , int day_removed[50]){
//evaluates user's input of daily report to their input of targets, and calculate the percentage of achievement
	int counter, i;
	int sum_BB, sum_Carbs, sum_Protein, sum_Fat, sum_Olahraga, sum_air, avg, tugas, step;
	
	//initialize variables as 0
	sum_BB = 0;
	sum_Carbs = 0;
	sum_Protein = 0;
	sum_Fat = 0;	
	sum_Olahraga = 0;
	sum_air = 0;
	counter = 0;
	
	//do the while looping as the currents value is filled
	while (current != NULL){
		
		sum_BB += current->BB;
		sum_Carbs += current->Carbs;
		sum_Protein += current->Protein;
		sum_Fat += current->Fat;
		sum_Olahraga += current->Olahraga;
		sum_air += current->Air;
		counter ++;
		
		current = current->next;
	}
	
	userptr->hari = counter;
	
	if(counter == 0){ //data checking as an error handling
		printf("Data masih kosong!");
	}
	
	else{  //displaying the evaluation 
		printf("\n\t\t=====================================");
		printf("\n\t\t==                                 ==");
		printf("\n\t\t== Evaluasi dietmu selama %d hari  == ", counter);
		printf("\n\t\t==                                 ==");		
		printf("\n\t\t=====================================");
				
		tugas = 0;
		step = 0;
		
		#pragma omp parallel private(tugas, step) //using parallel private to set tugas and step variable as private variables
		{
			#pragma omp master //using parallel master
			{
				for(i = step; i < 5; i++){  //distribute on a 5 sections of task
					tugas = i;
					step = i;
					#pragma omp task //task distribution to a thread
					{
						if (tugas == 0 && userptr->tBB != 0){
							float avg_BB = sum_BB / (float)counter;
							float result_BB = avg_BB /(float) userptr->tBB;
							#pragma omp critical //avoid race condition
							{
								printf("\n\n===========> BB ");
								printf("\nRata-rata berat badan : %.2f", avg_BB);
								printf("\nPersen ketercapaian target: %.2f %", result_BB * 100);
							}
						}
						if(tugas == 1 && userptr->tCarbs != 0){
							float avg_Carbs = sum_Carbs / (float)counter;
							float result_Carbs = avg_Carbs /(float)userptr->tCarbs;
							#pragma omp critical //avoid race condition
							{
								printf("\n\n===========> Karbohidrat ");
								printf("\nRata-rata konsumsi karbohidrat setiap harinya : %.2f", avg_Carbs);
								printf("\nPersen ketercapaian target: %.2f %", result_Carbs * 100);
							}
						}
						if(tugas == 2 && userptr->tProtein != 0){
							float avg_Protein = sum_Protein / (float)counter;
							float result_Protein = avg_Protein /(float)userptr->tProtein;
							#pragma omp critical //avoid race condition
							{
								printf("\n\n===========> Protein ");
								printf("\nRata-rata konsumsi protein setiap harinya : %.2f", avg_Protein);
								printf("\nPersen ketercapaian target: %.2f %", result_Protein * 100);
							}
						}
						if(tugas == 3 && userptr->tFat != 0){
							float avg_Fat = sum_Fat / (float)counter;
							float result_Fat = avg_Fat /(float)userptr->tFat;
							#pragma omp critical //avoid race condition
							{
								printf("\n\n===========> Lemak ");
								printf("\nRata-rata konsumsi lemak setiap harinya : %.2f", avg_Fat);
								printf("\nPersen ketercapaian target: %.2f %", result_Fat * 100);
							}
						}
						if(tugas == 4 && userptr->tOlahraga != 0){
							float avg_Olahraga = sum_Olahraga / (float)counter;
							float result_Olahraga = avg_Olahraga /(float)userptr->tOlahraga;
							#pragma omp critical //avoid race condition
							{
								printf("\n\n===========> Olahraga ");
								printf("\nRata-rata target Olahraga setiap harinya : %.2f", avg_Olahraga);
								printf("\nPersen ketercapaian target: %.2f %", result_Olahraga * 100);
							}
						}
						if(tugas == 5 && userptr->tAir != 0){
							float  avg_air = sum_air / (float)counter;
							float result_air = avg_air /(float)userptr->tAir;
							#pragma omp critical //avoid race condition
							{
								printf("\n\n===========> Air ");
								printf("\nRata-rata air yang diminum setiap harinya : %.2f", avg_air);
								printf("\nPersen ketercapaian target: %.2f %", result_air * 100);
							}
						}
					}						
				}
			}
		}
		#pragma omp taskwait //waiting for another thread to finish running a task
	}
}

void removeptr (Dietptr *startPtr, int day, int day_removed[50]){ 
	Dietptr prevPtr; //previous pointer before currentptr
	Dietptr tempPtr; //temporary pointer to insert
	Dietptr currentPtr; //pointer to current address
	
	int i;
	
	day++;
	
	//if the first day is entered then the first pointer will be removed
	if ( day == 0) { 
 		tempPtr = *startPtr;
		*startPtr = ( *startPtr )->next; 
 		free( tempPtr ); 
 	}
 	else {
 		prevPtr = *startPtr;
 		currentPtr = ( *startPtr )->next;
 		for(i = 1; i < day; i++){
 			if(day_removed[i]== 1){
 				continue;
			 }
 			if (i == day) {
		 		tempPtr = currentPtr;
		 		prevPtr->next = currentPtr->next;
		 		free( tempPtr );
		 	} 
 			prevPtr = currentPtr; 
 			currentPtr = currentPtr->next;
		 	if(currentPtr == NULL) {
			 	break;
			 }
 		}
 	}
}

void countPeriode(){
//calculates times needed to reach the weight target
	int BB, target;
	int temp, periode;		
	
	printf("\n\t\t=====================================");
	printf("\n\t\t==                                 ==");
	printf("\n\t\t==      Hitung Periode Target      == ");
	printf("\n\t\t==                                 ==");		
	printf("\n\t\t=====================================");
	
	printf("\n\n===========> Penurunan berat badan yang sehat yaitu 0.5kg perminggu atau 2kg perbulan");
	printf("\n<> Masukkan berat badanmu saat ini : ");
	scanf("%d", &BB);
	printf("\n<> Masukkan target berat badanmu : ");
	scanf("%d", &target);
	temp = BB-target;
	periode = temp/0.5;
	printf("\n<> Targetmu dapat tercapai dalam %d minggu jika dilakukan secara sehat dan konsisten!", periode);
	
	printf("\n\n");
	system("pause");
	system("cls");

}

void information(){
//contains informations about diet and KKB
	int i, pilihan, ntr;
	
	printf("\n\t\t=====================================");
	printf("\n\t\t==                                 ==");
	printf("\n\t\t==          Informasi              == ");
	printf("\n\t\t==                                 ==");		
	printf("\n\t\t=====================================");
	printf("\n\nInformasi yang ingin kamu ketahui :");
	printf("\n1. Daftar Nutrisi");
	printf("\n2. Diet Cutting & Diet Bulking \n  =  ");
	scanf("%d", &pilihan);
	
	if(pilihan == 1){
		printf("Pilih sumber nutrisi : \n1. Karbohidrat\n2. Protein\n3. Lemak\n");
		scanf("%d", &ntr);
		switch(ntr){
			case 1:
				printf("\t>> Karbohidrat  (per 100 gram)");
				printf("\n\n[1] Nasi :");
				printf("\na. nasi putih \t\t= 28 gram\nb. nasi merah \t\t= 23 gram");
				printf("\nc. nasi hitam \t\t= 34 gram\nd. nasi shirataki \t= 2.7 gram");
				printf("\n\n[2] Kentang \t\t= 17 gram");
				printf("\n\n[3] Jagung \t\t= 25 gram");
				printf("\n\n[4] Umbi");					
				printf("\na. ubi ungu/jalar/kuning \t= 20 gram");
				printf("\nb. singkong \t\t\t= 38 gram ");	
				printf("\n\n[5] Oat \t\t= 67 gram");
				break;
			case 2:
				printf("\t>> Protein  (per 100 gram)");
				printf("\n\n[1] Telur \t\t= 13 gram / butir");
				printf("\n\n[2] Tahu \t\t= 8 gram");
				printf("\n\n[3] Tempe \t\t= 19 gram");
				printf("\n\n[4] Ikan :"); 
				printf("\na. salmon \t\t= 20 gram");
				printf("\nb. tuna \t\t= 28 gram");
				printf("\nc. kod \t\t\t= 18 gram");	
				printf("\nd. tenggiri \t\t= 19 gram");							
				printf("\n\n[5] Susu \t\t= 3.4 gram");
				printf("\n\n[6] Daging sapi \t= 26 gram");
				printf("\n[7] Dada Ayam \t\t= 31 gram");				
				break;
			case 3:
				printf("\t>> Lemak  (per 100 gram)");
				printf("\n\n[1] Nabati :");
				printf("\na. alpukat \t\t= 15 gram");
				printf("\nb. biji-bijian \t\t= 49 gram");
				printf("\nc. kelapa \t\t= 33 gram");	
				printf("\nd. dark chocolate \t= 43 gram");
				printf("\ne. chia seed \t\t= 31 gram");
				printf("\n\n[2] Hewani :");
				printf("\na. minyak ikan \t\t= 15 gram");
				printf("\nb. keju \t\t= 33 gram");
				printf("\nc. susu \t\t= 1 gram");	
				printf("\nd. yogurt \t\t= 0.4 gram");				
				break;							
		}
	}
	else{
		#pragma omp parallel //parallel programming
 		{
    		int tid;
    		tid = omp_get_thread_num();
    	
			if(tid == 0){
				printf("\n\n===========> Diet Cutting yaitu pengaturan pola makan dengan memaksimalkan");
				printf("\npengurangan lemak sambil mempertahankan massa otot.");
				printf("\nKategori BMI yang disarankan : Normal - Obesitas");
			}
			if(tid == 1){
				printf("\n\n===========> Diet Bulking yaitu pengaturan pola makan yang berfokus pada kenaikan berat badan");
				printf("\nyaitu memaksimalkan pembentukan otot dengan makanan padat nutrisi dan kalori.");
				printf("\nKategori BMI yang disarankan : Underweight");
			}	
		}		  
	}
	printf("\n\n");
  	system("pause");
 	system("cls");
}

void help(){
//contains guide to use the app

	system("cls");
	printf("\n\t\t=====================================");
	printf("\n\t\t==                                 ==");
	printf("\n\t\t==             Bantuan             ==");
	printf("\n\t\t==                                 ==");		
	printf("\n\t\t=====================================");
	
  #pragma omp parallel //parallel programming
  {
    int tid;
    tid = omp_get_thread_num();

    if(tid == 0){
	    printf("\n<> Akses informasi seputar diet dan IMT sebelum mengatur target");
		printf("\n<> Pastikan anda mengisi target sebelum mengakses fitur evaluasi dan laporan harian");

    }
    if(tid == 1){
    	printf("\n\n");
    	printf("\n<> Mohon untuk tidak memberikan input huruf agar tidak terjadi maka restart program");
    	printf("\n<> Pastikan anda selalu menekan -1 untuk mengakhiri program supaya data anda tersimpan");
    }
  }
  
  printf("\n\n");
  system("pause");
  system("cls");
}

//file handling functions
int file_user_read (User *userptr, int day_removed[50]){
	FILE *fptr;
	//read mode of file userDiet.txt
	fptr = fopen("userDiet.txt", "r");
	
	char temp;
	
	if(fptr == NULL){ //if the file is not exist, create the file
		fclose(fptr);
		fptr = fopen("userDiet.txt", "w"); //write mode of file userDiet.txt
		fclose(fptr);
		return 0;
	}
	else{	
		fseek(fptr, 0, SEEK_SET);
		fscanf(fptr, "%c", &temp);
		fscanf(fptr, "%[^\n]", &userptr->nama);
		fscanf(fptr, "\n%d", &userptr->umur);		
		fscanf(fptr, "\n%d", &userptr->target_status);
		fscanf(fptr, "\n%d", &userptr->tBB);
		fscanf(fptr, "\n%d", &userptr->tCarbs);	
		fscanf(fptr, "\n%d", &userptr->tProtein);
		fscanf(fptr, "\n%d", &userptr->tFat);			
		fscanf(fptr, "\n%d", &userptr->tOlahraga);
		fscanf(fptr, "\n%d", &userptr->tAir);	
		fscanf(fptr, "\n%d", &userptr->hari);
	}
	fclose(fptr); //close file
	return 1;
}

int file_user_write (User *userptr, int day_removed[50]){
	
	FILE *fptr;
	fptr = fopen("userDiet.txt", "w"); //write mode of file userDiet.txt

	fprintf(fptr, "Printed Data :");
	fprintf(fptr, "\nNama : \t%s", userptr->nama);
	fprintf(fptr, "\nTarget berhasil disimpan! (%d)", userptr->target_status);
	fprintf(fptr, "\nTarget berat badanmu : \t%d kg", userptr->tBB);
	fprintf(fptr, "\nTarget karbohidrat harianmu : \t%d gram", userptr->tCarbs);
	fprintf(fptr, "\nTarget protein harianmu : \t%d gram", userptr->tProtein);
	fprintf(fptr, "\nTarget lemak harianmu : \t%d gram", userptr->tFat);
	fprintf(fptr, "\nTarget olahraga harianmu : \t%d menit", userptr->tOlahraga);
	fprintf(fptr, "\nTarget air minum harianmu : \t%d liter", userptr->tAir);
	fprintf(fptr, "\nJumlah hari yang kamu input : \t%d hari", userptr->hari);
	
	fclose(fptr); //close file
}

int file_target_write (Dietptr current, int day_removed[50]){
	
	FILE *fptr;
	//write on the file Target.txt
	fptr = fopen("Target.txt", "w");
	while (current != NULL){
		
		fprintf(fptr,"\n<> Berat badanmu hari ini %d kg",current->BB);
		fprintf(fptr,"\n<> Konsumsi karbohidrat hari ini %d gram",current->Carbs);
		fprintf(fptr,"\n<> Konsumsi protein hari ini %d gram",current->Protein);
		fprintf(fptr,"\n<> Konsumsi lemak hari ini %d gram",current->Fat);
		fprintf(fptr,"\n<> Kamu melakukan %d menit olahraga",current->Olahraga);
		fprintf(fptr,"\n<> Konsumsi airmu hari ini %d liter",current->Air);
		
		current = current->next;
	}
	
	fclose(fptr);
}

int file_target_read (Dietptr *sptr, int day_removed[50], int i, int *posisi){
	
	FILE *fptr;	
	
	int BB_var, Carbs_var, Olahraga_var,  air_var;
 
	Dietptr currentptr; //pointer to current address
	Dietptr newptr; //new pointer to insert
	Dietptr prevptr; //previous pointer before currentptr
	
	newptr = malloc(sizeof(Diet));
	
	//open the file in the first iteration to check whether the file has been created or not
	if(i == 0){
		fptr = fopen("Target.txt", "r");
	}
	if(fptr == NULL){
		fclose(fptr);
		fptr = fopen("Target.txt", "w");
		fclose(fptr);
		return 0;
	}
	else{ 
		if(i == 0){ //if the file already exists and in the first iteration the position will be set to the beginning of reading the file
			fseek(fptr, 0, SEEK_SET);
		}
		else{ //if the file already exists and the iteration is not the first then the position will be set to the last position
			fseek(fptr,  0 , SEEK_CUR);
		}
	
	//scan and insert the value of variables
		fscanf(fptr,"\n%d",&newptr->BB);
		fscanf(fptr,"\n%d",&newptr->Carbs);
		fscanf(fptr,"\n%d",&newptr->Protein);
		fscanf(fptr,"\n%d",&newptr->Fat);
		fscanf(fptr,"\n%d",&newptr->Olahraga);
		fscanf(fptr,"\n%d",&newptr->Air);
	
		newptr->next = NULL;
		prevptr = NULL;
		currentptr = *sptr;
	
		while(currentptr != NULL){
			prevptr = currentptr;
			currentptr = currentptr->next;
		}
		if (prevptr == NULL){
			newptr->next = *sptr;
			*sptr = newptr;
		}
		else{
			prevptr->next = newptr;
			newptr->next = currentptr;
		}
		if(i == -1){
			fclose(fptr);
		}	
		return 1;
	}
}

int file_removed_write(int hari[50]){ 
//remove a day of user's daily report in write mode
	FILE *fptr;
  
	fptr = fopen("removeday.txt", "w");
	
	int i;
  
	for(i = 0; i < 50; i++){
		fprintf(fptr, "\n%d", hari[i]);
	}
	fclose(fptr);
	
}


int file_removed_read(int hari[50]){
//remove a day of user's daily report in read mode
	FILE *fptr;
	fptr = fopen("removeday.txt", "r");
	
	if (fptr == NULL){
		fclose(fptr);
		fptr = fopen("removeday.txt", "w");
		fclose (fptr);
		return 0;
	}
	
	else{ 
		int i;
		for(i = 0 ; i < 50; i++){
			fscanf(fptr, "\n%d", &hari[i]);
		}
		return 1;
	}	
}

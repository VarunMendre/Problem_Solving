void Vector(){
    vector <int>v ;

    v.push_back(1);
    v.emplace_back(2); // emplace_back is faster than push_back

    vector<pair<int,int>>vec;

    v.push_back({1,2});
    v.emplace_back(3,4);



    //accessing elemets  : iterators 

    vector<int>::iterator it v.begin();       

    it++;
    cout<<*(it) << " "; //accessing the elemet 

    vector<int>::iterator it = v.end()

    //printing elements

    1:
     for(vector<int>::iterator it = v.begin() ; it != v.end();it++){
        cout<<*(it)<<" ";
    }
    2 :
    for(auto it = v.begin();it != v.end();it++){
        cout<<*(it);
    }
    3 : 
    for(auto it : v){
        cout<<it << " ";
    }


    //{10,20,30,40}
    v.erase(v.begin() + 1);   //20 will be erase

    {10,20,12,23,35}
    v.erase(v.begin()+2 , v.begin()+4);  //{10,20,35} 


    //insert
    vector<int>v(2,100); =>//{100,100}
    v.insert(v.begin(),300) =>//{300,100,100}
    v.insert(v.begin()+1,2,10) => //{300,10,10,100,100}

    vector<int>copy(2,50);
    v.insert(v.begin(),copy.begin(), copy.end()) // => {50,50,300,10,10,100,100}

}

void List(){
    
}
void stack(){
    stack<int>st;

    st.push(10);
    st.push(30);
    st.push(40);

    cout<<st.top();   // all operation are in O(1)
}
void Queue{

}
void priorityQueue{
         push => log n
         top => O(1)
         pop => log n
}

void set{
     //Sorted order , unique elements

     set<int>st;

     st.inser()     every thing are in Log n

}
void multiset{
    sorted but no unique
}
void unorderd_set{
    unique elements 
}

void map{
    key -> value
    unique ->  
    key would be any datatype

    map<int,int>mpp;

    map<int,pair<int,int>>mpp;

    map <pair<int,int>,int>mpp;

    mpp[1] = 2;    => {1,2}
    mpp.insert({2,3})  => {2,3}

    for(auto it : mpp){
        cout<<it.first<<" "<<it.second<<endl;
    }


}
void multimap(){
    //only it can store multiple keys
    //only map[key] cannot be used here
}


//algorithms 

sorting 

array : 
 sort(a , a+ n)

vector : 
sort(v.begin(), v.end()); 


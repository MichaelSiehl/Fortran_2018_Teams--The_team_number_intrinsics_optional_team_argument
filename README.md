# Fortran_2018_Teams--The_team_number_intrinsics_optional_team_argument
Fortran 2018 teams: Test case showing practical use of the `team_number` intrinsic function's optional team argument

# Overview and Test Case
The Fortran 2018 intrinsic function `team_number` does provide an optional team argument. The simple example program below is intended as an example of it's usefulness in practice.<br />

If we execute the same `form team` statement on all images within a `change team` construct (that already comprises a set of teams), that single `form team` statement does not only form a single set of new teams but instead can form multiple sets of new teams at once:<br />
For each team in MainTeam a new set of teams is created in SubTeam; Since MainTeam comprises 3 teams, and the `form team` statement in SubTeam does form 2 new teams, that single `form team` statement does actually form 6 new teams.<br />
With such nested teams, the team argument should allow to use the `team_number` intrinsic to get information about the nesting and the belonging of the images of a nested team with respect to their parent team. This could be helpful for the development of new kinds of parallel algorithms later on.

```fortran
program Main
! please compile and run with 15 coarray images
  !
  use,intrinsic :: ISO_FORTRAN_ENV, only: team_type
  implicit none
  !
  integer :: intTeamNumber
  type (team_type) :: MainTeam
  type (team_type) :: SubTeam
  !
!
!************************************************
! MainTeam **************************************
!************************************************
!
! split the 15 images of the initial team into 3 child teams consisting of 5 images each:
  if (this_image() <= 5) then
    intTeamNumber = 1
  else if (this_image() >= 6 .and. this_image() <= 10) then
    intTeamNumber = 2
  else if (this_image() >= 11) then
    intTeamNumber = 3
  end if
  !
form team (intTeamNumber, MainTeam) ! this creates the 3 child teams in MainTeam
!
change team (MainTeam)
!
! important: the following gets executed on the images of all the 3 newly created child teams:
!
!************************************************
! SubTeam ***************************************
!************************************************
!
! split the 5 images of the MainTeam's child teams into two distinct teams resp.:
  if (this_image() < 3) then
    intTeamNumber = 1
  else
    intTeamNumber = 2
  end if
!
form team (intTeamNumber, SubTeam) ! this generates 6 teams (2 new sub teams in the 3 main teams resp.)
                                   ! within the change team (MainTeam) construct
change team (SubTeam)
!
SubTeam_select: select case (team_number())
!
case (1) SubTeam_select
  ! codes for child team 1 of SubTeam:
  write(*,*) 'image number: ', this_image(), ' of team number: ', team_number(), &
             ' in SubTeam, of MainTeam''s child team number:', team_number(MainTeam)
  !
!*******************
case (2) SubTeam_select
  ! codes for child team 2 of SubTeam:
  write(*,*) 'image number: ', this_image(), ' of team number: ', team_number(), &
             ' in SubTeam, of MainTeam''s child team number:', team_number(MainTeam)
!
end select SubTeam_select
!
end team !(SubTeam)
!
!*************
!*************
!*************
!
end team !(MainTeam)
!
end program Main
```
